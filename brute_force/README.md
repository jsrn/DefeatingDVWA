# Brute Force

This challenge focuses on brute-forcing a generic username/password log on panel. An initial attempt to log in with random details yields the response:

```
Username and/or password incorrect.
```

Well alright, that gives us something to work with. Before diving in, take a while to familiarise yourself with [Burp Intruder](http://portswigger.net/burp/help/intruder.html), which is what I used. 

For now we'll keep it simple. Try to log in with the username `admin` and password `admin`. We're not lucky enough for that to be correct, so take a look at the `HTTP History` subsection of the `Proxy` tab. You should be able to find the GET request to	`/dvwa/vulnerabilities/brute/?username=whatever&password=whatever&Login=Login`. Right click on it, and select `Send to Intruder`.

We'll keep it simple and attempt to brute force the password for the `admin` user.

Remove all payload positions except for the `password` parameter in the GET request. On the payloads tab, under `Payload Options`, you can import your password list. I have provided one for this web app. You can find it [here](files/passwords.txt).

Now that we've loaded our password list into Burp and set the payload position, selecting `Intruder -> Start attack` on the top menu will open the attack window and begin sending requests.

This is great, but all we have is a bunch of requests. It's not immediately clear which request, if any, gets us any closer to having access. Under the options, we can add keywords to flag any response containing that response. Since we know an incorrect login attempt will spit out "Username and/or password incorrect." we can add "incorrect" to the list.

## Low

Having done the previous preparation, we can now just relaunch the attack. There is now a column called "incorrect". As the responses roll in, we see almost every box is ticked. When we get down to "password", however. There is no tick!

If we try logging in with the combination `admin`/`password`, we are greeted with the following:

```
Welcome to the password protected area admin
```

A look at the page source code also reveals something interesting.

```php
$user = $_GET['username'];
    
$pass = $_GET['password'];
$pass = md5($pass);

$qry = "SELECT * FROM `users` WHERE user='$user' AND password='$pass';"; 
```

Another case of non-existent input sanitisation.

Attempting to log in with the username `admin;#` escapes the rest of the query and saves us the effort of having to brute force at all.

## Medium

```php
// Sanitise username input
$user = $_GET[ 'username' ];
$user = mysql_real_escape_string( $user );

// Sanitise password input
$pass = $_GET[ 'password' ];
$pass = mysql_real_escape_string( $pass );
$pass = md5( $pass );
```

We're not so lucky this time! They've fixed the SQL injection vulnerability by cleaning their inputs. However, using burpsuite for the brute force is still equally successful.

## High

Under high security, DVWA adds an artificial delay of three seconds after each incorrect password. If you're lucky enough to have a target with a weak password, this still makes brute force more or less infeasible. You can escape this problem by using a tool that allows you to send requests in parallel. Sadly, the free edition of Burp does not.