### Woocomerce Currency Symbol Change
```php
add_filter('woocommerce_currency_symbol', 'change_existing_currency_symbol', 10, 2);

function change_existing_currency_symbol( $currency_symbol, $currency ) {
     switch( $currency ) {
          case 'LKR': $currency_symbol = 'Rs'; break;
     }
     return $currency_symbol;
}
```
