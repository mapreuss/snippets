# Template editables at design section

## Functions.php

```php
/* Customizer functions */
//adding setting for copyright text
add_action('customize_register', 'sumone_customizer');

function sumone_customizer($wp_customize) {
    //adding section in wordpress customizer   
    $wp_customize->add_section('splash_options_section', array(
        'title'          => 'Splash area'
    ));

    // Title
    $wp_customize->add_setting('title_content', array(
        'default'        => 'Estamos revolucionando o varejo',
    ));

    $wp_customize->add_control('title_content', array(
        'label'   => 'Main title text',
        'section' => 'splash_options_section',
        'type'    => 'text',
    ));
    
    // Background Img
    $wp_customize->add_setting('bg_img', array(
        'default'        => 'http://localhost:8888/wp/wp-content/uploads/2017/06/splash.png',
    ));
	  $wp_customize->add_control(
       new WP_Customize_Image_Control(
           $wp_customize,
           'logo',
           array(
               'label'      => __( 'Upload a logo', 'theme_name' ),
               'section'    => 'splash_options_section',
               'settings'   => 'bg_img',
               'context'    => 'your_setting_context' 
           )
       )
   );
}
```

## Theme
```html
<header class="splash" style="background: url(<?php echo get_theme_mod('bg_img'); ?>) 0 0 no-repeat;">

<h1><strong><?php echo get_theme_mod('title_content'); ?></strong></h1>
```

