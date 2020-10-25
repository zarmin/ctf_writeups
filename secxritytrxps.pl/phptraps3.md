# PHPTraps 3 / 3

```
<?php
if(isset($_POST['input']) && isset($_POST['way']))
{
    $input = (string)$_POST['input'];
    $way = (string)$_POST['way'];
    switch($way)
    {
        case '1' :
            $input = str_replace('LL','',$input);
            if(strpos($input, 'MAWEKLSFINALCHALLENGE') !== false)
                echo solve();
            else
                echo 'Fail.';
            break;
        case '2' :
            if(preg_match('/^.*(FINAL).*$/s', $input))
                $input = 'FAIL';
            if(strpos($input, 'MAWEKLSFINALCHALLENGE') !== false)
                echo solve();
            else
                echo 'Fail.';
            break;
        case '3' :
            $input = substr($input, 0, 26);
            $input = str_replace('MAWEKL','',$input);
            if(strpos($input, 'MAWEKLSFINALCHALLENGE') !== false)
                echo solve();
            else
                echo 'Fail.';
    }
}
?>
```

## Solution

The keyword is `pcre.backtrack_limit`

PHP / PCRE is matching from end to front.

If the backtrace limit is too low (or the input is too high), it will return `false`. Which will bypass the `$input = 'FAIL'` check.

```
# python3
'MAWEKLSFINALCHALLENGE' + 'a' * (1024*1024)
```



