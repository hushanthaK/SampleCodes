```php

add_action('rest_api_init', function () {
  register_rest_route('custom-registration-api/v1', '/register', array(
      'methods'  => 'POST',
      'callback' => 'register_user_callback',
  ));
  register_rest_route('custom-registration-api/v1', '/otp-login', array(
      'methods'  => 'POST',
      'callback' => 'login_otp_user_callback',
  ));
  register_rest_route('custom-registration-api/v1', '/verify-otp', array(
      'methods'  => 'POST',
      'callback' => 'verify_otp_user_callback',
  ));
});

// User register with mobile number
function register_user_callback($request) {
    $parameters = $request->get_json_params();

    // API params - username, email, country_code, mobile_number
    $username = sanitize_text_field($parameters['username']);
    $email = sanitize_email($parameters['email']);
	  $country_code = sanitize_text_field($parameters['country_code']);
    $mobile_number = sanitize_text_field($parameters['mobile_number']);
    Generate user password
    $password = wp_generate_password();

    // Get user by mobile number
		$user = get_users(array('meta_key' => 'mobile_number', 'meta_value' => $mobile_number));

    // Check user exist
		if (count($user) > 0) {
        return new WP_Error('invalid_user', 'Mobile number already exists.', array('status' => 400));
    }

    // Generate and send OTP to the mobile number
    $otp = generate_otp();
    send_otp_sms($mobile_number, $otp, $country_code);

    // Create new user
    $user_id = wp_create_user($username, $password, $email);

    if (is_wp_error($user_id)) {
        $error_message = $user_id->get_error_message();
        return new WP_Error('registration_failed', $error_message, array('status' => 400));
    }

    // Store the OTP in user meta data
    update_user_meta($user_id, 'mobile_number', $mobile_number);

    // Ths lines requred for digit plugin - if it use
	  update_user_meta($user_id, 'digits_phone', $country_code.$mobile_number);
    update_user_meta($user_id, 'digt_countrycode', $country_code);
    update_user_meta($user_id, 'digits_phone_no', $mobile_number);

    // Set woocomerce biling phone
    update_user_meta($user_id, "billing_phone", $mobile_number);

    // for custom field for save number
	  update_user_meta($user_id, "registered_phone_number", $mobile_number);

    // save OTP
    update_user_meta($user_id, 'otp', $otp);

    return true;
}

// User login with mobile number
function login_otp_user_callback($request) {
	$parameters = $request->get_json_params();

  // API params - country_code, mobile_number
  $mobile_number = sanitize_text_field($parameters['mobile_number']);
	$country_code = sanitize_text_field($parameters['country_code']);

	// Check if the mobile number is provided
  if (empty($mobile_number)) {
      return new WP_Error('invalid_input', 'Mobile number is required.', array('status' => 400));
  }

  // Check user exist
	$user = get_users(array('meta_key' => 'mobile_number', 'meta_value' => $mobile_number));
	
	if (!$user) {
    return new WP_Error('invalid_user', 'Invalid mobile number.', array('status' => 400));
  }

	// Generate and send OTP to the mobile number
  $otp = generate_otp();
  send_otp_sms($mobile_number, $otp,$country_code);
	

  // Get user ID
  $user_id = $user[0]->ID;
  //update OTP
	update_user_meta($user_id, 'otp', $otp);

  return true;
}

// Check user OTP is valid
function verify_otp_user_callback($request) {
	$parameters = $request->get_json_params();

  // API params - mobile_number, top
  $mobile_number = sanitize_text_field($parameters['mobile_number']);
  $otp = sanitize_text_field($parameters['otp']);
	
  // Check if the mobile number and OTP are provided
  if (empty($mobile_number) || empty($otp)) {
    return new WP_Error('invalid_input', 'Mobile number and OTP are required.', array('status' => 400));
  }

  // Get the user ID based on the mobile number
	$user = get_users(array('meta_key' => 'mobile_number', 'meta_value' => $mobile_number));
  if (!$user && !$user[0]) {
    return  new WP_Error('invalid_user', 'Invalid mobile number.', array('status' => 400));
  }
  $user_id = $user[0]->ID;

  // Get saved otp
	$saved_otp = get_user_meta($user_id, 'otp', true);
  // Verify the OTP
  if ($otp !== $saved_otp) {
    return new WP_Error('invalid_otp', 'Invalid OTP.', array('status' => 400));
  }
	// Remove the OTP from user meta data after successful verification
  //delete_user_meta($user_id, 'otp');

  // Generate authentication token for the user
  $expiration = time() + 365 * DAY_IN_SECONDS;
  $cookie = wp_generate_auth_cookie($user_id, $expiration, 'logged_in');
  //$token = wp_generate_auth_cookie($user_id, 86400);
	
	$response['wp_user_id'] = $user_id;
    $response['cookie'] = $cookie;
    $response['user_login'] = $user[0]->user_login;
    $response['user'] = getResponseUserInfo($user[0]);

    return $response;
}

// Generate OTP
function generate_otp() {
    // Generate a 6-digit random OTP
    return sprintf("%06d", mt_rand(1, 999999));
}

function send_otp_sms($mobile_number, $otp, $country_code) {
    // Code to send SMS with the OTP to the provided mobile number using your preferred SMS service provider
    // Implement the logic to send the OTP via SMS
    $api_key = 'bUdGPXVGeUlITkNDSWdhcUlncW0=';
	if (substr($mobile_number, 0, 1) === '0') {
    	$mobile_number = $country_code . substr($mobile_number, 1);
    } else {
		$mobile_number = $country_code.$mobile_number;
		$url = 'https://sms.ipsova.com/sms/api?action=send-sms&api_key=' . $api_key . '&to=' . $mobile_number . '&from=Quickee&sms=' . 'MSG=Your OTP code is ' . $otp;
		$response = wp_remote_post($url);
		if (is_wp_error($response)) {
        	$error_message = $response->get_error_message();
        	return;
    	}
	}
}

function getResponseUserInfo($user)
{
    $shipping = null;
    $billing = null;
    $avatar = get_user_meta($user->ID, 'user_avatar', true);
    if (!isset($avatar) || $avatar == "" || is_bool($avatar)) {
        $avatar = get_avatar_url($user->ID);
    } else {
        $avatar = $avatar[0];
    }
    $is_driver_available = false;
    //if(is_plugin_active('delivery-drivers-for-woocommerce/delivery-drivers-for-woocommerce.php')){
  //$is_driver_available = true;
//}
    return array(
        "id" => $user->ID,
        "username" => $user->user_login,
        "nicename" => $user->user_nicename,
        "email" => $user->user_email,
        "url" => $user->user_url,
        "registered" => $user->user_registered,
        "displayname" => $user->display_name,
        "firstname" => $user->user_firstname,
        "lastname" => $user->last_name,
        "nickname" => $user->nickname,
        "description" => $user->user_description,
        "capabilities" => $user->wp_capabilities,
        "role" => $user->roles,
        "shipping" => $shipping,
        "billing" => $billing,
        "avatar" => $avatar,
        "is_driver_available" => $is_driver_available,
        "dokan_enable_selling" => null
    );
}

```
