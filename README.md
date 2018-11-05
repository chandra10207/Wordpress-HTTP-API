# Wordpress HTTP API
Localize Script:
Add this on your functions.php
```php
//enqueue script
wp_enqueue_script( 'my-custom-js', get_theme_file_uri( '/js/custom.js' ), array( 'jquery' ), '1.0', true );

//localize script
wp_localize_script('my-custom-js', 'myobject', array(
    'ajax_url'          => admin_url('admin-ajax.php'),
    'home_url'            => home_url(),
));
```


Add your action:
Add this on your functions.php
```php
add_action('wp_ajax_api_result', 'get_api_result');
add_action('wp_ajax_nopriv_api_result', 'get_api_result');
function get_api_result()
{
	global $wpdb, $product, $post;
	$url = $_POST['website_url'];
	$request = wp_remote_get( 'https://pippinsplugins.com/edd-api/products' );
	//$request = wp_remote_get( 'https://sitecheck.sucuri.net/results/google.com' );

 if( is_wp_error( $request ) ) {
 	echo 'Hello';
 	return false; // Bail early
 }
print_r($request);
$body = wp_remote_retrieve_body( $request );
//print_r($body);
$data = json_decode( $body );
$returndata = '';
//print_r($data);
if( ! empty( $data ) ) {
	$returndata.= '<ul>';
	foreach( $data->products as $product ) {
		//echo '<li>';
		$returndata.= '<li>';
		$returndata.= '<a href="' . esc_url( $product->info->link ) . '">' . $product->info->title . '</a>';
		$returndata.= '</li>';
	}
	$returndata.= '</ul>';
}
//$data = "Hello world";
echo $returndata;
wp_die();
}
```
API Form Shortcode:
Add this on your functions.php. Use [api_check] shortcode.
```php

add_shortcode( 'api_check', 'api_check_function' ); 
function api_check_function( $atts ) {
    ?> 
    <form id="api-form" method = "post">
        <input id="website_url" type="text" name="website-url">
        <input id="api-form-submit" type="submit" name="gg">
    </form>

    <div id="result" class="api-result"></div>

<?php
}
```

Javascript:
Add this on your custom.js
```Javascript
jQuery( document ).ready(function($) {

    $('#api-form-submit').click(function(e){
        e.preventDefault();
        var webUrl = $('#website_url').val();
        alert(webUrl);

        jQuery.ajax({
                type: 'POST',
                data: {
                    'action': 'api_result',
                    'website_url': webUrl,
                },
                url: myobject.ajax_url, 
                success: function(data, lable) {
                    $("#result").html(data);
                },
                error: function(data) {
                    $("#result").text("Data not found");
                    
                }
            });


    })

   
});
```

