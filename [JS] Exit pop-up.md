CSS

```css
#exitPopUp {
    position: fixed;
    background: rgba(0,0,0,.3);
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    z-index: 999999999999;
    display: none;
}
.closePopUp{
    position: absolute;
    display: block;
    width: 25px;
    height: 25px;
    border-radius: 50%;
    background: gray;
    color: white;
    text-align: center;
    line-height: 25px;
    font-weight: bold;
    font-size: 15px;
    right: 0;
    top: 0;
    cursor: pointer;
}
#exitPopUp ul{
    list-style: none;
    margin: 0;
    padding: 0;
    position: absolute;
    top: 25%;
    left: 25%;
}
#exitPopUp li{
    position: relative;
    display: none;
    width: 714px;
    height: 461px;
}
```css

PHP

```php
<div id="exitPopUp">
    <?php $new_query = new WP_Query( array(
        'post_type'      => rich_material
    ) );
    ?>
    <ul>
        <?php
        while ( $new_query->have_posts() ) : $new_query->the_post();  
        $image = get_post_meta( get_the_ID(), 'rich_material_shareimg', true );
        $url = get_post_meta( get_the_ID(), 'rich_material_url', true );
        ?>
            <li><a href="<?php echo $url;?>" target="_blank"><img src="<?php echo $image;?>"></a> <a href="#" class="closePopUp">X</a></li>
        <?php
        endwhile;  
        wp_reset_postdata();?>
    </ul>
</div>
```

JS

```js
//random exit-popup
var total = jQuery("#exitPopUp li").length;
var number = Math.floor((Math.random() * total) + 1);
jQuery("#exitPopUp").find("li:nth-child("+number+")").show();

// Fechar pop-up
jQuery(".closePopUp").on("click", function(e){
    e.preventDefault;
    jQuery("#exitPopUp").hide();
});

// mostrar pop-up
var showpopup = function(){
    // Mostrar só depois da pessoa estar na página por 5 segundos
    setTimeout(function(){ 
        // e a pessoa sair da página
        var $window = jQuery(window),
            $html = jQuery('html');
        $window.on('mouseleave', function(event) {
            console.log(event.target);
            if (!$html.is(event.target))
                return;
            jQuery('#exitPopUp').show();
        });                
    },5000);		
}


// exibir depois de quantos dias?
var numberOfDaysToAdd = 6;

// que dia é hoje?
var today = new Date().getTime() + (30 * 24 * 60 * 60 * 1000)

var future = new Date().getTime() + (30+numberOfDaysToAdd)*24*60*60*1000;

// verifica se existe alguma data guardada
var showedTime = localStorage.getItem("popupTime");


// se sim
if(showedTime !== null){

    // verifica se já passou tempo suficiente desde a última exibição
    showedTime = parseInt(showedTime);
    if (today > showedTime){
        // Mostrar pop-up
        showpopup();
        // Atualizar data no localstorage
        localStorage.setItem("popupTime", future);			
    }
} else {
    // a pop-up nunca foi mostrada antes, então vamos criar a data 
    localStorage.setItem("popupTime", future);

    // Mostrar pop-up
    showpopup();
}
```
