---
title: "THM Pickle Rick"
date: 2021-07-29T14:02:39+07:00
draft: false
comments: false
images:
author: "Sarel"
---

***This writeup goes over finding all three ingredients in the [TryHackMe Pickle Rick](https://tryhackme.com/room/picklerick) room, without obtaining a shell***

## Background

There are three flags that we need to get.
For this writeup, the IP of the victim is 10.10.79.234

## Enumeration

An nmap scan is run to determine the ports and services available on the machine

```
nmap -sV 10.10.79.234
```
The results show that SSH is open, and that there is a webserver.

![nmap results](https://i.imgur.com/reM5uXS.png)

The website is live, but has little information on the front end

![website](https://i.imgur.com/C4sSJjt.png)

Checking the page source of the website, reveals a comment containing a username.

![username](https://i.imgur.com/iG6xiZu.png)

If we have a username, it means there should be a login page. This can be enumerated with gobuster. The command used is[^1]:

```
gobuster -m dir -u http://10.10.79.234 -w /usr/share/wordlists/dirb/common.txt -x php,txt,html
``` 

![gobuster](https://i.imgur.com/S02uxQf.png)

Two results that stand out are login.php and robots.txt

Looking at `http://10.10.79.234/robots.txt` simply shows the line *"Wubbalubbadubdub"*

Heading to `http://10.10.79.234/login.php` presents us with a login page.

![login](https://i.imgur.com/ySm9EPl.png)

## Exploitation

As a wild guess, we enter the username we found earlier and the phrase we found in the robots.txt file. We are able to successfully log in and are shown a command panel.

![command](https://i.imgur.com/GC3F6Fl.png)

We can try to enter `ls` as a command, and receive an output.

![ls](https://i.imgur.com/r4arqkY.png)

It works, so we can assume that other linux commands will also work.

---

### The First Ingredient

The ingredient is right there, and we can try to open it with the command `cat Sup3rS3cretPickl3Ingred.txt`, however, it fails. Certain commands are disabled.

![disabled command](https://i.imgur.com/KYAGeL4.png)

However, if we try to use `less` instead of `cat`, we get an output containing the first ingredient.
```less Sup3rS3cretPickl3Ingred.txt```

![less](https://i.imgur.com/RgO7EuR.png)

---

### The Second Ingredient

Using `less clue.txt` reveals that the other ingredients are on the file system.

![clue](https://i.imgur.com/H33PVo3.png)

We can try navigating the rest of the system. `ls /home` shows two user directories.

![home](https://i.imgur.com/ujBD4De.png)

We can see what is inside Rick's directory with `ls /home/rick`.

![rick](https://i.imgur.com/jTPaST1.png)

This reveals the second ingredient, which we can see with `less /home/rick/"second ingredients"`.

![second](https://i.imgur.com/srkWCF5.png)

---

### The Third Ingredient

We may assume that the final ingredient may be in the root folder, however we are not able to access it as is. We run the `sudo -l` command to see if we can run any commands as sudo that can be used to our advantage.

![sudo](https://i.imgur.com/o8wQjcg.png)

It seems like our user has unrestricted access to the `sudo` command. Thus, in order to view the content of the root folder, we can use `sudo ls /root`.

![root folder](https://i.imgur.com/tB8Ttw6.png)

Root does indeed contain the final ingredient, which can be viewed with `sudo less /root/3rd.txt`

![third](https://i.imgur.com/67eM0rs.png)

## Final Notes

This writeup only explored the simplest, newbie friendly path of exploitation. The only knowledge needed was [nmap](https://tryhackme.com/room/furthernmap), [gobuster](https://tryhackme.com/room/webenumerationv2), and some [basic linux commands](https://tryhackme.com/room/linuxfundamentalspart1). 

There are many ways to gain the flags, including using the unrestricted sudo access to create a reverse shell.

---

[^1]: This command lets gobuster scan in "directory" mode. We use a wordlist from dirb (an application similar to gobuster), because gobuster on Kali does not have its own wordlist. The `-x` option allows us to only search for certain file extensions.