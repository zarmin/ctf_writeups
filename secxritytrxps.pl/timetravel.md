# Timetravel

`get.php`
```
<?php
setcookie('id',uniqid());
?>
```

`eat.php`
```
<?php
$cookie=$_POST['cookie'];
session_save_path('/home/mawekl/timetravel/');
session_start();
echo 'You ate: '.htmlspecialchars($cookie);
echo "\n<br>";
$_SESSION['cookie']=$cookie;
$_SESSION['time']=time();
?>
```

`check.php`
```
<?php
$cookie=$_COOKIE['id'];
$time=time();
session_save_path('/home/mawekl/timetravel/');
session_start();
if ($cookie!=$_SESSION['cookie'])
    die('Wrong cookie');
if ($time+20!=$_SESSION['time'])
    die('You must eat cookie after 20 seconds from now, but you ate it '.($time-$_SESSION['time']).' seconds ago');
echo $_PASSWORD;
?>
```

To solve it, it requires 3 different PHP "feaures" at once.

- file based session storage locks the session file, while the PHP script is running
- PHP scripts can be paused when flushing a large output buffer and the client can't receive it fast enough
- PHP loose comparsion will mark this true: "0e555555555555" == "0e0" 

The last one is required, because the cookie must be at maximum 4096 bytes, and we need to send a larger payload and htmlspecialchars length expansion (like " -> &quot;) is not enough.

## The plan

- send an eat query with large payload (`cookie=0e5........`) and a ~20-21 sec timer
- send a check query after cookie id of `0e0`
- try to catch the correct timing, or may adjust it with a few seconds / fractions

## Timeline

- eat query sent and start printing output for the 'You ate:' echo line until the output buffer is full
- check query is started, saves the `time` in a local variable and blocked, because the file based session storage
- waiting 20 seconds to receive the output buffer for eat query
- eat query's results is received `$_SESSION['time']` is set to a current time.
- check query is unblocked, checking the current session time, with a locally stored value from 20 secs before.
- PASSWORD is printed, challenged is solved!

