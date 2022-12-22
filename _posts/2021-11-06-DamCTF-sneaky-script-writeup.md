---
title: DamCTF - sneaky-script - Writeup
tags: damctf malware
---

# Description

![2021-11-06_16-24-36](https://user-images.githubusercontent.com/84577967/140614866-795e6e02-759e-4322-85fe-6517bc0879b6.png)

This challenge was part of a recent CTF [DamCTF](https://damctf.xyz/), and it was the only challenge in the malware category.

# Solution

Upon extracting the provided zip file, we found that it contained two files: a pcap file and a bash script.

![2021-11-06_16-42-44](https://user-images.githubusercontent.com/84577967/140615375-62bf876d-6ee8-4f83-83eb-051c9c99b517.png)

Examining the bash script, I found a line of code that indicates that the script is downloading and executing a Python file from a remote host.

![2021-11-06_16-20-48](https://user-images.githubusercontent.com/84577967/140615442-3020d258-13cd-4cbb-988a-f7fa3714f7d5.png)

Although the host mentioned in the bash script appears to be down, we can still examine the pcap file to see if it contains any useful information.
By analyzing the captured traffic in the pcap file, we were able to find the base64 encoded file that was downloaded and executed by the bash script.

![2021-11-06_16-20-19](https://user-images.githubusercontent.com/84577967/140615524-cae32560-bb31-400e-ba73-280f338c8ce8.png)

We can use a base64 decoder to decode the file and then use the `file` tool to identify the type of file.

![2021-11-06_16-48-20](https://user-images.githubusercontent.com/84577967/140615571-5f638027-cfd4-4b8a-b8e2-ced6a0ff8392.png)

A python3.6 byte-compiled file, we can use [uncompyle6](https://github.com/rocky/python-uncompyle6/) to decomplie the file.

![2021-11-06_16-23-12](https://user-images.githubusercontent.com/84577967/140615661-19c0c02c-a1a5-4464-a783-1a0a93cbfbdf.png)

Upon analyzing the Python file, we found that it was designed to collect sensitive information such as `network, process list, ssh keys ...`, and send it to a new host at the endpoint `/upload`.

![2021-11-06_16-23-32](https://user-images.githubusercontent.com/84577967/140616016-873d16b8-bd62-4271-85ff-eb0c74c67032.png)

The data that was transmitted by the Python file was encrypted using the XOR algorithm and the key `8675309`. 
To decrypt this data, we can save it to a file and run a Python script to apply the decryption process using the known key.

```python
import base64

def decrypt():
	r = ""
	with open('encrypted.txt') as f:
		contents = f.read()
		p = base64.b64decode(contents)
		k = b'8675309'
		for i in range(len(p)):
			d = p[i] ^ k[i % len(k)]
			r += chr(d)
		print(r)
decrypt()
```
# Flag

The flag:

![2021-11-06_17-06-20](https://user-images.githubusercontent.com/84577967/140616098-fba25eb7-0526-44b4-9f48-11243985a779.png)
