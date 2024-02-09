// Remove "Select options" button from (variable) products on the main WooCommerce shop page.
add_filter( 'woocommerce_loop_add_to_cart_link', function( $product ) {

 	global $product;

	$classes = implode(
        ' ',
        array_filter(
            array(
                'button',
                wc_wp_theme_get_element_class_name( 'button' ), // escaped in the template.
                'product_type_' . $product->product_type,
                $product->is_purchasable() && $product->is_in_stock() ? 'add_to_cart_button' : '',
                $product->supports( 'ajax_add_to_cart' ) && $product->is_purchasable() && $product->is_in_stock() ? 'ajax_add_to_cart' : '',
            )
        )
    );
	
	    $attributes = array(
        'data-product_id'  => $product->get_id(),
        'data-product_sku' => $product->get_sku(),
        'aria-label'       => $product->add_to_cart_description(),
        'aria-describedby' => $product->add_to_cart_aria_describedby(),
        'rel'              => 'nofollow',
    );

	if ( 'variable' === $product->product_type ) {
// 		return '123';
		$classes = str_replace("product_type_variable", "", $classes);
	} 
// 	else {
		return sprintf( '<a href="%s" data-quantity="%s" class="%s" %s>%s</a>',
			esc_url( $product->add_to_cart_url() ),
			esc_attr( isset( $args['quantity'] ) ? $args['quantity'] : 1 ),
			esc_attr( $classes ),
			wc_implode_html_attributes( $attributes ),
			esc_html( $product->add_to_cart_text() )
		);
// 	}

} );
