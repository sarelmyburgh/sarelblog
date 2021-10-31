---
title: "THM Net Sec Challenge"
date: 2021-10-31T08:05:07+07:00
draft: false
comments: false
images:
author: "Sarel"
---

***This Write up is for the [Net Sec Challenge](https://tryhackme.com/room/netsecchallenge) room, which is a part of the Jr Pentester Path of TryHackMe***

## Background

This room tests your nmap, hydra and telnet abilities and although it is classified as "Medium", the room is pretty easy if you paid attention to the rest of the rooms in the [Network Security](https://tryhackme.com/module/network-security) module.

## Enumeration

In order to simplify our workflow, I will save the target IP as a variable with `export ip=10.10.102.108`. Now we can simply use `$ip` whenever we want to enter the full IP address.

A simple nmap scan with the `-sV` switch already enables us to answer *five* of the questions in one go! Note how Question 2 mentions a port above 10,000, so perhaps we can use the `-p-` switch as well. So our final command will be:
```sudo nmap -sV -p- $ip```

![nmap results](https://imgur.com/upPEFuA.png)

From that one scan we can answer all of the following questions:

- What is the highest port number being open less than 10,000?
- There is an open port outside the common 1000 ports; it is above 10,000. What is it?
- How many TCP ports are open?
- What is the flag hidden in the SSH server header?
- We have an FTP server listening on a nonstandard port. What is the version of the FTP server?

Now we only need three more answers.

## HTTP Header

If you paid attention in the [Protocols and Servers](https://tryhackme.com/room/protocolsandservers) room, you can use Telnet to receive an HTTP header

```telnet $ip 80```

This  will open the connection, after which we need to request the page with:

```GET / HTTP/1.1```

and set the host

```host: telnet```

We will then receive the header containing the flag.

![http flag](https://imgur.com/BCWHJWq.png)

This challenge could also have been completed using Burp Suite instead of Telnet.

## Cracking passwords

The next question we can attempt is the ftp passwords for "eddie" and "quinn". This can be accomplished with Hydra. We can either crack them one by one, similar to how it was done in the [Protocols and Servers 2](https://tryhackme.com/room/protocolsandservers2) room, or if we already know (or read up on) Hydra, we can create a userlist which contains both those usernames. Note that, on this server, ftp is not on the standard port and our command needs to reflect that.

```hydra -L netsecusers.txt -P /usr/share/wordlists/rockyou.txt ftp://$ip:10021```

This gives us the passwords of both users.

![hydra](https://imgur.com/I2T0gtN.png)

Once you have the passwords, there are multiple ways to access the files. In this example, I simply pointed my browser to ftp://10.10.102.108:10021 and was promted for a username and password. testing both, you will see that Quinn has the flag. Download and view the file to answer question 7.

![ftp](https://imgur.com/VulaDNV.png)

## Avoiding IDS

The last question asks us to open a website for another challenge. This turns out to be an IDS that checks our nmap scans. If you used the `-sV` scan earlier, yours should show 100% chance of being detected. Oops.

We can attempt different scan types to try and avoid the IDS. The `-sN` tag seems to be effective, so we can reset the packet count on the website and run another nmap scan as follows:

```sudo nmap -sN $ip```

This will trigger the flag on the website.

![ids](https://imgur.com/AmDzl6P.png)

## Final notes

This was a very easy room for which TryHackMe provides ample training in the module. It was interesting to see a simplified view of how the IDS can pick up on an nmap scan, as different kinds of options such as `-sS`, `-f` and even `-T 1` all got picked up, though some to a lesser extent than others.

I was already familiar with Hydra and Nmap before attempting this challenge, however learning to use Telnet to fetch HTTP headers in lieu of Burp Suite was new to me.
