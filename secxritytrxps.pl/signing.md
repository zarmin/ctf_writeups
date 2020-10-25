# Signing

`sign.php`
```
require 'keys.php';
$path = '/home/mxwxkl/public_html/challs/signing/';
$files = dir($path);
while($file = $files->read())
{
    if(substr($file,-4) != '.sql') continue;

    $file = $path.$file;
    $query = file_get_contents($file);

    $sign  = md5($salt.$query);
    $sign .= pack('L',crc32($sign));

    for($i=0;$i<strlen($sign);$i++)
        $sign[$i] = chr(ord($sign[$i]) ^ ord($key[$i]));
    $file = mcrypt_encrypt(MCRYPT_RIJNDAEL_128, substr($key,0,32), $file, MCRYPT_MODE_CBC, $iv);
    echo '<a href="application.php?path=' . urlencode(base64_encode($file));
    echo '&sign=' . urlencode(base64_encode($sign)) . '">Query</a><br/>';
}
```

`application.php`
```
<?php

/*
There are two passwords:
1) First password is in the .sql file.
2) Second password is in the database.
*/

function error()
{
    die("Error.");
}

require 'keys.php';

$path = base64_decode((string)$_GET['path']);
$sign = base64_decode((string)$_GET['sign']);
if(strlen($path) <= 8 || (strlen($path)%8) != 0)
    error();

$path = mcrypt_decrypt(MCRYPT_RIJNDAEL_128, substr($key,0,32), $path, MCRYPT_MODE_CBC, $iv);

#delete AES padding
for($i=0;$i<strlen($path);$i++)
    if($path[$i] == "\x00")
    {
        $path = substr($path,0,$i);
        break;
    }

for($i=0;$i<strlen($sign);$i++)
    $sign[$i] = chr(ord($sign[$i]) ^ ord($key[$i]));

$md5 = substr($sign,0,strlen($sign)-4);

$crc  = substr($sign,strlen($sign)-4,4);
$crc = unpack('L',$crc);
$crc = $crc[1];

if($crc != crc32($md5)) error();

$query = file_get_contents($path);

if(md5($salt.$query) != $md5)
    error();

if(stripos($query,'sleep') !== false || stripos($query,'benchmark') !== false)
    die('Please do not abuse my server ;)');

mysql_query(addslashes($query)) or die('Password is very close! Do not give up!');

echo 'Done.';

?>
```

## Intro

A lot of techniques are required to solve this challenge

- Basic cryptography knowledge
- AES CBC decryption manipulation
- Decryption oracles
- PHP `file_get_contents` RFC2397 method
- Hash length extension attack
- Error based blind SQL injection

## Solution, part I, getting the SQL filename

- Padding is custom, everything after the first null byte will be deleted
- The AES128 CBC output is not validated immediately (no integrity)
- Using an oracle between the signature validation and the content validation is the key. 

### Part1, finding an oracle

`file_get_contents` can be an oracle (if error display is disabled) if it crashes (reading too much memory) or hangs (http download timeout) the script.

- HTTP download can't be done, because we can't control the first block
- Reading large files can be promising, but there are no known large files in the relative path, but e.g. `/dev/zero` is a good source.

The encoded filepath's decrypted content is something like this:

```
/home/mxwxkl/pub
lic_html/challs/
signing/********
***.sql000000000
```

Split into 16 byte blocks (due AES 128).

- Algo is AES128 CBC
- IV is unknown
- Plaintext is known for 2 first blocks
- Ciphertext is known for all blocks

AES128 CBC using the following algo for first two blocks.
```
P0 = IV ^ S(C0, K)
P1 = C0 ^ S(C1, K)
```

- P0, P1: plaintext block 0 and 1
- C0, C1: ciphertext block 0 and 1
- IV: initialization vector
- S: AES128 decryption function
- K: key

We can set custom plaintext for block1 if we change the ciphertext for block0 (by xoring the original P1, orignal C0 with our chosen plaintext for block1, X1), it will also cause garbage output for plaintext block0.

`C0_new = P1 ^ C0 ^ X1`

The target is to have something like this:

```
/***************
/../dev/zero0***
****************
****************
```

- 0 is a nullbyte
- * is a dontcare
- first should contain only one / and no nullbytes

`file_get_contents` will use this path to load the infinite contents of `/dev/zero` and cause an out-of-memory termination in PHP.

To achive this we need to send a few tries to the server with calculated `C0_new` blocks to produce a block with `/***************` pattern.

If we found one, the script will fail with `500 Internal server error`, instead of `Error.`. Now we can try out custom `sign` arguments and if it's valid we will get an `Internal server error`.

### Part2, retrieving the key bytes

The `sign` arguments first 32 bytes are an output of a hex encoded md5, so the charset is `0-9a-f`.
This can be used to find out all of the possible keybytes for each index (16 possibilities to each one).

- Finding the first 4 bytes
  - a minimum of 4 bytes are required for a proper 32 bit length decoding `unpack('L', ...)` for the CRC32 
  - because of the previously determined keybyte possibilities, we need 16\*\*4 = 65536 tries
  - `crc32("") = 0`, packed into 32 bit integer, `00 00 00 00`. This is xored by the keybytes, so the right answer will directly reveal the keybytes.
- Finding the next 28 bytes (5..32nd bytes)
  - example for the 5th byte: 
    - `k`: key bytes
    - `b`: sent input bytes
    - `crc32(b0 ^ k0) = (b1 ^ k1, b2 ^ k2, b3 ^ k3, b4 ^ k4)`
    - `k4` is unknown and need to be figured out
    - sending in the known k0 byte will produce a `crc32("\x00") = 0xd202ef8d` output (because of the xor). The 0xd202ef8d output should be xored with k1..k3 known key bytes and the last byte can be cracked (16 possibilities).
  - by this example the further 27 byte can be cracked
- Finding the last 4 bytes (33..36th bytes)
  - same the previous ones, but these keybytes are for xoring the crc32, not the md5, which increases the keyspace to 256 possibilities per bytes

After this we have the keybytes.

### Part3, retrieving the SQL file name

By knowing the key bytes, we can run a AES128 CBC decrypt on the encrypted path name. We don't kwow the IV yet, but it is only affecting the first block (which we already know), so we can use full nullbytes as the IV.

The IV can be calculated by using a nullbyte only IV, getting the plaintext for block0 and xoring the plaintext with original plaintext block0 (`/home/mxwxkl/pub`).

### Part4, getting the password from the SQL file

The SQL file is accessible from the browser so it's no a big deal.

At this part we know:
- The key bytes for AES128 and signature xoring
- The IV
- The decrypted path and the sql file path
- The sql file contents

## Solution, Part II, getting the password stored in the SQL database

We need to able to pass a custom command to the mysql and exfiltrate the password via a blind, error-based mysql injection.

### Part1, custom file contents

`file_get_contents` can receive an RFC2397 encoded string like `data://text/plain;base64,<base64>`, which can be used to pass custom data

### Part2, bypass the salt based md5 signature check for the contents

`md5(secret || message)` based solutions are susceptible for [hash length extension attacks](https://en.wikipedia.org/wiki/Length_extension_attack).

For this we need to have:
- The original message (we have it)
- The original hash (we have it)
- The length of the secret (we don't know, but we can use an oracle to determine it)
- A message to append

With this we can calculate this hash output, without knowing the secret

`md5(secret || message || padding || appended_message)`

Also we need to forge a signature which is easy at this point
- crc32 the md5 output and pack it as a 32bit unsigned int
- xor the md5 output and the packed crc with the key bytes

So we can send a custom data appended to the original SQL file contents:
- calculate the padding bytes and the new signature hash by using the length extension method
- encode the full message as base64 and use the data (RFC2397) solution to pass it
- forge a signature

### Part3, exfiltrate the database contents

The original SQL file is looking like something like this:

```
SELECT password FROM sqli3 WHERE id=1337 -- Congratz! <password bytes>
```

We can append a new line and a OR/AND to the WHERE query. We will use an error based blind SQL injection:

```
" \n OR (select exp( ( ord(substr(password, " + pos + ", 1)) = " + char + " ) * 710) from sqli3 limit 1 )"
```

The two parameters are the `pos` and `char`.
- `pos`: the current position (run it until no new characters are found)
- `char`: the current tested character's ascii code (run it between 32 and 127)

If the current character matches, then `exp(710)` is evaluated, if not `exp(0)` is evaluated. `EXP` returns e raised to the power of the specified number, argument 710 will overflow the double's max value and cause an error. This will print "Password is very close! Do not give up!".

By this method we can extract all of the character so we're done.

