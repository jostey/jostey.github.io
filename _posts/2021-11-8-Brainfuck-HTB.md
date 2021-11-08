---
layout: post
title: Brainfuck - HackTheBox
slug: brainfuck-hackthebox
date: '2021-11-08T13:41.350Z'
---

![logo]({{ site.baseurl }}/images/brainfuck/logo.jpg)

# Info
Name: Brainfuck

OS: Linux

# Recon
Starting a port scan with nmap:
{% highlight shell %}
nmap -p- -sS -min-rate 5000 --open -vvv -n -Pn 10.10.10.17 -oN nmap/initial
{% endhighlight  %}
![nmap-lame]({{ site.baseurl }}/images/brainfuck/1.jpg)

* Port 22 -- OpenSSH 7.2p2 -- Username Enumeration
* Port 25 -- Postfix smtpd
* Port 110 -- Dovecot pop3d
* Ports 143 -- Dovecot imapd
* Ports 443 -- nginx 1.10.0

## Virtual domains
We can see into ssl-cert of the server Nginx that brainfuck.htb and sup3rs3cr3t.brainfuck.htb are a valid hostnames, so we will add them in /etc/hosts.

![nmap-lame-2]({{ site.baseurl }}/images/brainfuck/2.jpg)
![vir-dom]({{ site.baseurl }}/images/brainfuck/3.jpg)

Next, we can browse to the URL https://brainfuck.htb. It is a Wordpress site according the description of the website.
In the sup3rs3cr3t URL we can see a forum where only they can write: admin and orestis.
![user]({{ site.baseurl }}/images/brainfuck/4.jpg)

## User Enumeration
We have got a 2 possible users: admin and orestis.

