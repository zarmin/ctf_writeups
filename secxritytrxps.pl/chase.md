# Chase

```
<?php

function quit()
{
    global $solution;
    die('You did it o_0! Solution: ' . $solution);
}

function chase($a)
{
    foreach($a as $k => $v)
    {
        if(preg_match('/hidemeplease/',$k))
            die('WANTED!');
        if(is_array($v))
            chase($v);
        else
            if(preg_match('/hidemeplease/',$v))
                die('WANTED!!');
    }
}

function escape($a)
{
    foreach($a as $k => $v)
    {
        if(preg_match('/o_O quit is here!/',$k))
            quit();
        if(is_array($v))
            escape($v);
        else
            if(preg_match('/o_O quit is here!/',$v))
                quit();
    }
}

$s = urldecode($_SERVER['QUERY_STRING']);
if($s) {
    chase($_GET);

    if(!preg_match('/hidemeplease/', $s))
        die('WANTED!!!');

    if(preg_match('/o_O quit is here!/', $s))
        die('WANTED!!!!');
    
    escape($_GET);

    die('W A N T E D :(');
}
```

- `hidemeplease` string should be in urldecoded `QUERY_STRING`, but not in `_GET` tree.
- `o_O quit is here!` string shoudn't be in urldecoded `QUERY_STRING`, but in `_GET` tree.

solve the hidemeplase is easy:
`?a=hidemeplease&a=1`
- second key will overwrite the first one. `hidemeplease` will be `QUERY_STRING`, but not in `_GET` tree

solve the `o_O quit is here` part is harder.

approaches:
- `_GET` key naming conversions (will convert '.', ' ', '[' to dots) --> it will convert spaces too
- `_GET` can have a body --> PHP does not support it out of the box
- Register globals, GET variable override from POST / Cookie / Header --> nope, no RegGlobals enabled.
- `urldecode` works differently as a PHP function and on `_GET` params. --> could not find any differencies
- `QUERY_STRING` truncation on long strings? --> nope
- Some lower/uppercase mangling --> nope
- Wrongly formatted `QUERY_STRING` syntax. --> **YES IT IS!**

`?o[aaaa=b`
will become

```
_GET -> ["o_aaaa"]=> string(1) "b"
QUERY_STRING -> o[aaaa=b
```

So this will do the trick: `o[O quit is here=a`.

Strangely in this situation the `_GET` key can contain spaces without getting converted to underscores.

