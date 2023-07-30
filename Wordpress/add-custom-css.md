### Add Custom CSS to Wordpress Site
```php
    add_action('init', 'add_custom_css');

    function add_custom_css() {
      // add custom css code with your logics
      if(condition) {
       echo '<style type="text/css">
         .mobile-icon-bar {display:none !important}
       </style>';
      }

      /**************************** OR ****************************/

       echo '<style type="text/css">
         .mobile-icon-bar {display:none !important}
       </style>';
    }
```
