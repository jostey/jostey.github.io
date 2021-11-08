---
layout: post
title: Brainfuck - HackTheBox
slug: brainfuck-hackthebox
date: '2021-11-08T13:41.350Z'
---

| Name      	| OS    	|
|-----------	|-------	|
| Brainfuck 	| Linux 	|

# Recon
Starting a port scan with nmap:
{% highlight shell %}
IP=10.10.10.3
sudo nmap -p- -sS --min-rate 5000 --open -vvv -n -Pn $IP -oG nmap/initial
{% endhighlight  %}
![nmap-lame]({{ site.baseurl }}/images/lame/1.jpg)
