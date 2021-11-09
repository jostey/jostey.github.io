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

We can scan with nmap scripts for version and vulnerabilities associated with the ports mentioned above: 

{% highlight shell %}
sudo nmap -p22,25,110,143,443 -sCV 10.10.10.17 -oG nmap/targeted
{% endhighlight %}

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
In the sup3rs3cr3t.brainfuck.htb URL we can see a forum where only they can write: admin and orestis.


## User Enumeration
We have got a 2 possible users: admin and orestis.
![user]({{ site.baseurl }}/images/brainfuck/4.jpg)

# WordPress Vulns
It is a good idea to research vulnerabilities in WordPress. We can use wpscan (without TLS Checks):
{% highlight shell %}
wpscan --url https://brainfuck.htb --disable-tls-checks
{% endhighlight %}

![wpscan]({{ site.baseurl }}/images/brainfuck/wpscan.jpg)

This command return a lot of information about the page, but it is important to look for plugins with vulnerabilities, because they are the typical way to attack these systems. Searchsploit returns valuable data:

![searchsploit]({{ site.baseurl }}/images/brainfuck/searchsploit.jpg)

If we read the PoC of Privilege Escalation (41006 in searchsploit), we can see a HTML with a POST call to a domain.abc/wp-admin/admin-ajax.php . We should edit the value of username with a valid user. As we had an admin user we tried with him.

![exploit]({{ site.baseurl }}/images/brainfuck/exploit.jpg)
![admin]({{ site.baseurl }}/images/brainfuck/admin.jpg)

## Clue SMTP
We found a clue in the WordPress posts. We must look for more information in the SMTP service.
![clue-smtp]({{ site.baseurl }}/images/brainfuck/clue-smtp.jpg)

## WordPress Reverse Shell 404 Template
We can attack this machine with a reverse shell in PHP into the 404 error template. We can login into dashboard and edit the template. At first glance we will not see the template but it inherits from another theme (Specia).

We must edit the IP and port (if you want) with the php-reverse-shell (pentestmonkey): http://pentestmonkey.net/tools/web-shells/php-reverse-shell
![404-template]({{ site.baseurl }}/images/brainfuck/404-template.jpg)

Wrong way! "You need to make this file writable before you can save your changes."

# SMTP - Port 25
We can see the credentials of the user oretis in a plugin of WordPress called Easy WP SMTP:
orestis -- kHGuERB29DNiNE
![smtp-credential]({{ site.baseurl }}/images/brainfuck/smtp-credential.jpg)

