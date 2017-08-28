# Shortcodes in wordpress

More complete sample. 

Use in wordpress:
`[clients title="Sua marca não vai ser nem a 1ª, nem a 2ª…" strong="nem a 90ª a lucrar mais com a SumOne" number1="1300" label1="lojas" number2="90" label2="marcas" number3="7" label3="verticais"]`

Renders

![](https://image.prntscr.com/image/Vr8INA5ZR9yr698aFZlhBg.png)

## File

Create `resources/clients.php` file.

```php
<?php
// Create Shortcode clients
// Use the shortcode: [clients title="Sua marca não vai ser nem a 1ª, nem a 2ª…" strong="nem a 90ª a lucrar mais com a SumOne" number1="1300" label1="lojas" number2="90" label2="marcas" number3="7" label3="verticais"]
function create_clients_shortcode($atts) {
	// Attributes
	$atts = shortcode_atts(
		array(
			'thetitle' => '',
			'strong' => '',
			'number1' => '1300',
			'label1' => 'lojas',
			'number2' => '90',
			'label2' => 'marcas',
			'number3' => '7',
			'label3' => 'verticais',
		),
		$atts,
		'clients'
	);
	// Attributes in var
	$thetitle = $atts['thetitle'];
	$strong = $atts['strong'];
	$number1 = $atts['number1'];
	$label1 = $atts['label1'];
	$number2 = $atts['number2'];
	$label2 = $atts['label2'];
	$number3 = $atts['number3'];
	$label3 = $atts['label3'];	

    ob_start();
?>

<section class="clients">
	
    <div class="container">
        <article>
            <h3><?php echo ($thetitle); ?>
            <strong><?php echo($strong); ?></strong></h3>

            <ul class="bullets">
                <li><strong><?php echo($number1); ?></strong> <?php echo($label1); ?></li>
                <li><strong><?php echo($number2); ?></strong> <?php echo($label2); ?></li>
                <li><strong><?php echo($number3); ?></strong> <?php echo($label3); ?></li>
            </ul>
        </article>
    </div>

    <section class="listClients">
		<h3 id="titleClients">Alguns clientes que já estão fazendo a revolução no varejo</h3>
        <ul class="carousel" id="brands">
           <!-- content grabbed by js -->
        </ul>
    </section>

</section>

<?php return ob_get_clean();  }
add_shortcode( 'clients', 'create_clients_shortcode' ); ?>
```

## Functions.php

Add to functions php

```php
add_filter('widget_text', 'do_shortcode');
include ('resources/shortcodes/clients.php');
```



