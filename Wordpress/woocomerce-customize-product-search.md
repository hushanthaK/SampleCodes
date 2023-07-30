### Map query fields
```php
add_action('pre_get_posts', 'advanced_search_query', 1000);

function advanced_search_query($query) {
  if($query->is_search()) {
    $slug = '';
    if (isset($_GET['category'])) {

      if($term = get_term_by( 'id', $_GET['category'], 'product_cat' )){
        $slug = $term->slug;
      }
  
      $query->set('tax_query', array(array(
          'taxonomy' => 'product_cat',
          'field' => 'slug',
          'terms' => array($slug) )
      ));
    }
    return $query;
  }
}
```

### Customize Search Query
```php
add_filter( 'posts_search', 'mu_custom_search_query', 20, 2 );

function mu_custom_search_query( $search, $wp_query ) {
  global $wpdb;
  
  if ( empty( $search ) ) {
      return $search;
  }
  
  $q = $wp_query->query_vars;
  $n = !empty( $q['exact'] ) ? '' : '%';
  
  $search = $searchand = '';
  //var_dump($q['tax_query']);
  //exit;
  foreach ( (array) $q['search_terms'] as $term ) {
    $term = esc_sql( like_escape( $term ) );
  
    $search .= "{$searchand}($wpdb->posts.post_title REGEXP '{$term}')";
  
    $searchand = ' AND ';
  }
  
  if ( ! empty( $search ) ) {
    $search = " AND ({$search}) ";
    if ( ! is_user_logged_in() ) {
        $search .= " AND ($wpdb->posts.post_password = '') ";
    }
  }
  
  return $search;
}
```
