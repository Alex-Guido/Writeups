# Summary
The following ports were discovered to be open by a Nmap scan: 22, 80, 6379, and 10000.

10000, the standard port for running Webmin, a management control panel, was the port of interest. The Redis service, which was running on port 6379, is what gave me an initial foothold.

Redis is an open-source data structure tool that can be used as a cache, message broker, or distributed in-memory database. Redis shouldn't be publicly accessible to the internet since, if improperly configured, an attack could utilize SSH to connect straight into the victim server. This is the method I used to gain an initial foothold.

I was able to generate my own public-private key, write that key to a file, and then import that key into redis. While logged into redis, you can save that public key to **authorized_keys** file on the server. Finally, using that generated private key, SSH into the redis server.
 Post enumeration, a user named Matt was found, and this user created an id_rsa.bak file which contained his readable private SSH key. I took that key offline and cracked it with ssh2john and John the Ripper. Because Matt reuses his passwords, I was able to log into Webmin with his credentials. 

The running version of Webmin in this box is vulnerable to CVE-2019-12840, which any user authorized to the "Package Updates" module can execute arbitrary commands with root privileges via the data parameter to update.cgi. I used a module in Metasploit to execute this and root the box.

## Nmap
```bash
PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 46:83:4f:f1:38:61:c0:1c:74:cb:b5:d1:4a:68:4d:77 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDem1MnCQG+yciWyLak5YeSzxh4HxjCgxKVfNc1LN+vE1OecEx+cu0bTD5xdQJmyKEkpZ+AVjhQo/esF09a94eMNKcp+bhK1g3wqzLyr6kwE0wTncuKD2bA9LCKOcM6W5GpHKUywB5A/TMPJ7UXeygHseFUZEa+yAYlhFKTt6QTmkLs64sqCna+D/cvtKaB4O9C+DNv5/W66caIaS/B/lPeqLiRoX1ad/GMacLFzqCwgaYeZ9YBnwIstsDcvK9+kCaUE7g2vdQ7JtnX0+kVlIXRi0WXta+BhWuGFWtOV0NYM9IDRkGjSXA4qOyUOBklwvienPt1x2jBrjV8v3p78Tzz
|   256 2d:8d:27:d2:df:15:1a:31:53:05:fb:ff:f0:62:26:89 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIRgCn2sRihplwq7a2XuFsHzC9hW+qA/QsZif9QKAEBiUK6jv/B+UxDiPJiQp3KZ3tX6Arff/FC0NXK27c3EppI=
|   256 ca:7c:82:aa:5a:d3:72:ca:8b:8a:38:3a:80:41:a0:45 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIF3FKsLVdJ5BN8bLpf80Gw89+4wUslxhI3wYfnS+53Xd
80/tcp    open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-favicon: Unknown favicon MD5: E234E3E8040EFB1ACD7028330A956EBF
|_http-title: The Cyber Geek's Personal Website
|_http-server-header: Apache/2.4.29 (Ubuntu)
6379/tcp  open  redis   syn-ack ttl 63 Redis key-value store 4.0.9
10000/tcp open  http    syn-ack ttl 63 MiniServ 1.910 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
|_http-favicon: Unknown favicon MD5: 91549383E709F4F1DD6C8DAB07890301
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

## Information Gathering
### Port 80
Port 80 (Apache httpd 2.4.29) is open displaying this webpage below.



