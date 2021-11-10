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

# SMTP and POP3 - Port 25 and Port 1110
We can see the credentials of the user oretis in a plugin of WordPress called Easy WP SMTP:

orestis -- kHGuERB29DNiNE

![smtp-credential]({{ site.baseurl }}/images/brainfuck/smtp-credential.jpg)

In SMTP service we cannot login because it is disabled. Instead, we can try logging in on port 110 (POP3) with the same credentials:
{% highlight shell %}
telnet 10.10.10.17 110
user orestis
pass kHGuERB29DNiNE
list
retr 1
retr 2
{% endhighlight %}

![pop3-1]({{ site.baseurl }}/images/brainfuck/pop3-1.jpg)
![pop3-1.1]({{ site.baseurl }}/images/brainfuck/pop3-1.1.jpg)
![pop3-2]({{ site.baseurl }}/images/brainfuck/pop3-2.jpg)

# Login into Sup3rs3cr3t Forum

orestis -- kIEnnfEKJ#9UmdO

![forum]({{ site.baseurl }}/images/brainfuck/login-forum.jpg)

We can see 2 posts interesting. One, where they say about a SSH Key encrypted and other post we can see a URL encrypted. We can see a simple encryption, like Caesar Substitution and we have a sentence encrypted and plain-text (sign of orestis messages):

* Wejmvse - Fbtkqal zqb rso rnl cwihsf

* Qbqquzs - Pnhekxs dpi fca fhf zdmgzt

* Orestis - Hacking for fun and profit

First, we need to identify the type of cypher: https://www.boxentriq.com/code-breaking/cipher-identifier

The cypher is Vigenere and it uses a cipher based on a key that spans the length of the phrase (if it is not large enough). By having a plaintext phrase and an encrypted phrase we can find out the key with which it encrypts because it is possible to do the reverse operation (correctly ordering the characters that come out cyclically repeated because the key is shorter than the phrase to encrypt).

![key]({{ site.baseurl }}/images/brainfuck/fuckmybrain.jpg)

With a key to decrypt a string we could decrypt a possible ssh key URL. That's right!

![vigenere]({{ site.baseurl }}/images/brainfuck/vigenere.jpg)

## Crack id_rsa (ssh2john)
Next, we have a id_rsa but there is a passphrase that we do not know. We can break the passphrase with John The Ripper and ssh2john (because the tool needs it in that format to work correctly). Then, with the list rockyou.txt we will crack the passphrase.

{% highlight shell %}
wget https://raw.githubusercontent.com/magnumripper/JohnTheRipper/bleeding-jumbo/run/ssh2john.py
python ssh2john.py id_rsa > id_rsa_hash
{% endhighlight %}

![ssh2john]({{ site.baseurl }}/images/brainfuck/ssh2john.jpg)

{% highlight shell %}
john --wordlist=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt id_rsa_hash
{% endhighlight %}

![john]({{ site.baseurl }}/images/brainfuck/john_cracked.jpg)

* Passphrase: 3poulakia!

# User Flag
With the passphrase it is trivial:
![user]({{ site.baseurl }}/images/brainfuck/user.jpg)

# Root Flag -- Priv Escalation (lxd/lxc Group)
With Linpeas.sh we can see a vuln about lxd/lxc Group. We have taken the following guide to scale and get the flag we can do it by searching a file called root.txt (method 2 without connection):

https://github.com/carlospolop/hacktricks/blob/master/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation.md#lxdlxc-group---privilege-escalation

![lxd]({{ site.baseurl }}/images/brainfuck/lxd.jpg)
