---
title: No WSL After 2 Day
tags: windows wsl linux terminal system
---

# Description

`No WSL After 2 Day !` OR `No Windows System Linux After Today !`

> This document explains how to install a virtual Linux system on your Windows machine without relying on WSL.

# Final result

![2021-12-14_15-52-47](https://user-images.githubusercontent.com/84577967/174202000-65a8e1ad-b4fb-4127-9bc1-31a5ac17382f.png)

# Download

Below are the software and application names mentioned in this document:

[Windows Terminal Preview](https://www.microsoft.com/store/productId/9N8G5RFZ9XK3)

[VirtualBox](https://www.virtualbox.org/wiki/Downloads)

[Kali Linux](https://www.kali.org/get-kali/#kali-virtual-machines)

# Setting our virual machine using VirtualBox

After downloading and installing Windows Terminal and VirtualBox, the next step is to extract the Kali Linux OVA Image file and import it into VirtualBox. It is not necessary to modify any settings at this stage; simply leave everything as the default settings.

![2021-12-14_16-49-33](https://user-images.githubusercontent.com/84577967/174202013-4f41c965-9263-4e49-92f5-1ceeb33dd627.png)

![2021-12-14_17-46-38](https://user-images.githubusercontent.com/84577967/174202021-91e05065-e104-4ba6-a7e1-434ac53d2cf3.png)

Once the image has been successfully imported, proceed to modify some machine settings by right-clicking on the machine, selecting 'Settings,' and following the steps shown in the accompanying screenshots below

1) In the advanced tab let's choose Bidirectional for both of `Shared Clipboard` & `Drag'nDrop`

![2021-12-14_18-22-37](https://user-images.githubusercontent.com/84577967/174202035-d7f98079-abc6-4095-ba4d-bf54ae6df8c0.png)

2) We recommend allocating 2 CPUs and 2 GB of RAM for the system's base memory and processor.

![2021-12-14_18-26-39](https://user-images.githubusercontent.com/84577967/174202045-238f8363-3fc8-45bc-bb99-f4b443592680.png)

![2021-12-14_18-26-52](https://user-images.githubusercontent.com/84577967/174202055-f62db662-793c-4f9f-82c3-7f54e9e2025b.png)

3) For the network, select 'NAT,' as this is a required configuration.

![2021-12-14_18-28-52](https://user-images.githubusercontent.com/84577967/174202071-68be61ab-3bfa-4fa0-b4b7-76e6650c680d.png)

4) Share a folder from your windows to the linux machine, In my case I shared my WIN desktop and mounted to `/root/Desktop/Windows/`. Be sure to select the `Make Permanent` option to ensure the folder remains accessible.

![2021-12-14_18-30-38](https://user-images.githubusercontent.com/84577967/174202080-a14d2dcd-840b-422a-a007-b4dfc3912ce1.png)

After that you set everything click OK and Ð congrats! Your machine is ready.

Great ! Now let's run our virtual machine in normal mode ...

![2021-12-14_18-36-38](https://user-images.githubusercontent.com/84577967/174202095-9ec46f35-df40-4d40-ad91-9851bb80528e.png)

Default creds: kali/kali

![2021-12-14_18-40-09](https://user-images.githubusercontent.com/84577967/174202107-98a4c016-798c-4bd5-aa62-ae4caa6b145f.png)

We need this to check only if our VM is working fine and also to check the network settings.

![2021-12-14_18-44-16](https://user-images.githubusercontent.com/84577967/174202154-04de206f-d53a-4c50-b269-305ca7962783.png)

Now generate a new ssh key for any user, I always like to be root.

So by going to terminal and type `sudo su -` then `ssh-keygen` and pressing `Enter` few times to generate the key, this will automatically save both the private and public keys in `/root/.ssh/id_rsa`

![2021-12-14_18-52-07](https://user-images.githubusercontent.com/84577967/174202168-c82c4034-53d5-4ecb-b17e-2ace5b0ffba8.png)

After that let's copy `id_rsa.pub` to `authorized_keys` by typing: `cp id_rsa.pub authorized_keys`, then changing the file permissions using chmod to set the access mode to 600 `chmod 600 authorized_keys`.

And then copy the private key `cat /root/.ssh/id_rsa`

![2021-12-14_18-51-41](https://user-images.githubusercontent.com/84577967/174202276-599a0069-1f97-4f76-999b-76360b4ff874.png)

And paste it into your Windows Desktop Directory

![2021-12-14_19-21-39](https://user-images.githubusercontent.com/84577967/174202256-6ad3889f-d284-4e1e-9dcd-329cc2db2dfd.png)

It's important to also change the permissions of the id_rsa file in Windows.

Please save the code in a file with a .cmd or .bat extension, and then execute it.

```
@echo Changing id_rsa Permission
icacls .\id_rsa /inheritance:r
icacls .\id_rsa /grant:r "%username%":"(R)"
@pause
```

Awesome! Now let's add this line to our `C:\Windows\System32\drivers\etc\hosts` file

```
127.0.0.1 kali.box
```

Return to `VirtualBox` and access the settings for the Kali machine. Navigate to the 'Network' category and click on 'Advanced'

![2021-12-14_19-03-25](https://user-images.githubusercontent.com/84577967/174202309-c0eb20bc-903b-4143-a35b-7bd78de778bf.png)

Next, select 'Port Forwarding'

![2021-12-14_19-03-54](https://user-images.githubusercontent.com/84577967/174202327-ce8c8383-08af-42ec-8714-66ef55626892.png)

And follow the steps in this screenshot -

![2021-12-14_19-06-29](https://user-images.githubusercontent.com/84577967/174202336-8a50f67f-8f5f-4314-887d-18cd51d649a2.png)

This rule enables direct connection from the Windows machine to the SSH port of the Kali VM.

Return to the machine and modify the sshd_config file to allow root logins `nano /etc/ssh/sshd_config`

![2021-12-14_19-45-27](https://user-images.githubusercontent.com/84577967/174202353-5fb724f9-d040-4eb3-9061-f99eba920dd3.png)

Let's change this line to

```
PermitRootLogin yes
```

After modifying the sshd_config file, restart the SSH service and enable it if it was not enabled by default.

![2021-12-14_20-11-16](https://user-images.githubusercontent.com/84577967/174202372-b1a33e59-1a5f-4a03-9d6c-c0070c0a2084.png)

Commands: 

```bash
# To restart SSH
systemctl restart ssh

# To enable SSH
systemctl enable ssh
```

# Setting our Windows Terminal

Now after that everything is set in our Kali virtual machine ... Let's go to the Windows Terminal and create a new profile

![2021-12-14_20-25-07](https://user-images.githubusercontent.com/84577967/174202383-e622ecdb-825d-4473-8d99-809eda9a2021.png)

Now follow the steps shown in the screenshot below

![2021-12-14_20-26-49](https://user-images.githubusercontent.com/84577967/174202395-4d0a6555-ceaf-4c2e-afdc-795969f9095b.png)

The command line:
```
ssh.exe -i id_rsa root@kali.box
```

**Note: `Starting directory` must be where we have our SSH `id_rsa` private key.**

After you set everything as shown click 'Save' and then go to startup and choose your new profile as the Default profile . Don't forget to click on 'Save'. 

![2021-12-14_20-32-52](https://user-images.githubusercontent.com/84577967/174202412-68c4f5c1-22f2-4b74-8197-9504a77e4586.png)

After restarting the Windows Terminal, it will automatically run the SSH command and connect you to your Kali VM.

![2021-12-14_20-35-28](https://user-images.githubusercontent.com/84577967/174202425-bde379e1-9612-414d-b987-bd6d55ed463f.png)

type `yes` and click Enter 

![2021-12-14_21-30-12](https://user-images.githubusercontent.com/84577967/174202433-d6da16a3-47c8-4e3e-a747-6f29cd7c873a.png)

To open the shell directly on our shared Desktop, we can add in the end of the file `~/.zshrc` the mounted point (path) we set earlier

```bash
cd /root/Desktop/Windows
# or
cd ~/Desktop/Windows
```

![2021-12-14_21-32-12](https://user-images.githubusercontent.com/84577967/174202457-19fcfafa-846b-4df9-a2c6-49096a4ce47b.png)

---

# Make it run at startup and headless

Go to `C:\Users\Username\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`, replace the value 'Username' with your computer username and create a new file. Let's name it `kali_vbox.bat`

`"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm "Kali-Linux-2021.4-vbox-amd64" --type headless`

First check if `VBoxManage.exe` is in the right path and the `"Kali-Linux-2021.4-vbox-amd64"` value we can get from the box Name in the settings.

![2021-12-14_21-43-12](https://user-images.githubusercontent.com/84577967/174202470-c8906b00-0fa4-4a71-9bd8-59544f21c584.png)

Congratulations! The Linux VM will now run in the background and you can use everything on your Windows desktop as if you were working on Linux.

![2021-12-14_21-37-02](https://user-images.githubusercontent.com/84577967/174202479-33a0edae-a420-4eb8-915d-bad3ce10dfc2.png)

--

# Additional Port Forwarding in my own configuration

![2021-12-14_19-08-49](https://user-images.githubusercontent.com/84577967/174202494-3a951b7c-ee8f-43f6-a307-f155058aac04.png)

The first IP is the IP address of my THM VPN, which I connect to using my Windows machine. I use this rule when playing a machine in THM or HTB, but I need to update the IP each time if I am not using a VIP membership.

The 2nd IP `10.0.2.15` is the default IP address for the NAT network adapter.

---

# Thanks

Finally, I would like to thank `leeloo1313` for the encouragement and help in writing this setup guide.

---

If you liked this content and want to encourage me for more, you can [buymeacoffee](https://www.buymeacoffee.com/ab2pentest).
