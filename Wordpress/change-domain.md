UPDATE wp_options SET option_value = replace(option_value, 'https://wordpress.org', 'https://wordpress.com') WHERE option_name = 'home' OR option_name = 'siteurl';

UPDATE wp_posts SET guid = replace(guid, 'https://wordpress.org','https://wordpress.com');

UPDATE wp_posts SET post_content = replace(post_content, 'https://wordpress.org', 'https://wordpress.com');

UPDATE wp_postmeta SET meta_value = replace(meta_value,'https://wordpress.org','https://wordpress.com');
