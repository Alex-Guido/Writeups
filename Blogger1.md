# Blogger: 1
This is a vulnerable machine from vuln hub.

[Blogger: 1 from vuln hub](https://www.vulnhub.com/entry/blogger-1,675/)

## Netdiscover scan

![Untitled](https://user-images.githubusercontent.com/55252902/163292638-140cb8fc-0083-40c3-a424-04b99b000b66.png)

## Nmap scan

![nmap scan](https://user-images.githubusercontent.com/55252902/163292720-dd822472-d199-40dc-b1d8-361101a105f9.JPG)

## Enumeration

From the nmap service scan I can see that port 80 is open and the version being run from the nmap service scan - Apache httpd 2.4.18 ((Ubuntu)).

![Untitled](https://user-images.githubusercontent.com/55252902/163292832-17376e5a-6211-4be4-aaaa-3d866344d367.png)

### Nikto scan I ran

![Untitled 1](https://user-images.githubusercontent.com/55252902/163292898-c1f7ad15-5332-422a-aac0-417425a3449d.png)

### Directory discovery through ffuf

![Untitled 2](https://user-images.githubusercontent.com/55252902/163292942-6f3be4ac-2c60-49ae-81ee-7c3c67285a57.png)
![Untitled 3](https://user-images.githubusercontent.com/55252902/163292965-9b69d4af-e5eb-47db-913e-29d3ff58e537.png)

I Went through all of these and the only interesting directory was fonts/ when I went into the fonts directory I found another directory named blog/.

![Untitled 4](https://user-images.githubusercontent.com/55252902/163292997-5dc8f2ca-98fe-446d-a59f-9373455b636c.png)
![Untitled 5](https://user-images.githubusercontent.com/55252902/163293003-a981b095-e431-4189-b056-ed296e9c9489.png)

Looks like a wordpress site. Another interesting thing I see here is j@m3s, seems like a user, I will take note of this.

I don’t know too much about vulnerabilities related to wordpress, so at this point I had to do some research and found a tool on Kali called wpscan.

I read up on the documentation and looked up a cheatsheet for wpscan.

`wpscan --url http://10.0.2.5/assets/fonts/blog/ --plugins-detection aggressive`

Ran this command.

![Untitled 6](https://user-images.githubusercontent.com/55252902/163293197-09a9ae4d-413f-481e-ad41-7139f82a12c8.png)
![Untitled 7](https://user-images.githubusercontent.com/55252902/163293209-8c11876b-45e2-42f6-b6aa-82a56a97521e.png)

Wordpress version 4.9.8 
Two plugins discovered


- Askimet is spam filtering, running an out of date version.


- wpDiscuz is a wordpress comment plugin that is also running an out of date version.


I couldn’t access any of the blogs, but the creator of this machine said to add this VM to etc/hosts, after I did that I could access post.

#Exploitation

![Untitled](https://user-images.githubusercontent.com/55252902/163293319-2411062f-5e6e-4712-a95c-88508c7ea882.png)
![Untitled 1](https://user-images.githubusercontent.com/55252902/163293339-e8921c88-c435-41be-b5b1-5e97bec3455f.png)
At the end of every post there is a comment section. This comment section allows you to attach files.

When I did a search for vulnerabilities I found that this version of wpDiscuz allows for unauthenticated RCE. I attempted this exploit https://www.exploit-db.com/exploits/49967, by trying to upload, but I got an error saying “this file type is not support.”

Through some more searching I found a video on youtube of how to exploit this.

[Video from Youtube](https://www.youtube.com/watch?v=h5nJp3RyiHw)

![youtubevideo](https://user-images.githubusercontent.com/55252902/163293553-0aa2cc97-c4f7-4a83-9409-4f04e07e5fc2.JPG)

Apparently, wpDiscuz allows for image attachments in comments which are uploaded to the site and also included in the comments. Only image attachments are allowed, but in this version of wpDiscuz this file type verification can be bypassed. Filetypes can be spoofed.

Before anything, I set up a listener. I am going to upload a PHP reverse shell.

I got it from here [Github - pentestmonkey / php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

Be sure to edit the IP and PORT variable. This part is important, but at the top of this code add GIF89a. This will trick the upload thinking this file is a gif by adding the gif file header at the top.

![Untitled 2](https://user-images.githubusercontent.com/55252902/163293614-9f7321a0-c693-4a10-810e-c0742b8fbf55.png)
![Untitled 3](https://user-images.githubusercontent.com/55252902/163293822-6d59c27d-47ed-4f84-b1ae-0b5dbba6b494.png)

File uploaded succesfully and a reverse dumb shell is obtained. 

![Untitled 4](https://user-images.githubusercontent.com/55252902/163293851-2a85841e-590f-4ee4-b1d4-324ae6579a7d.png)

I am not root on this machine, so let’s see what can be found. Snooping through the home directory these seem to be users. I Went through all of the users but james has a user.txt file that can’t be accessed.

![Untitled 5](https://user-images.githubusercontent.com/55252902/163293890-76a7b347-8710-46a8-a2cb-5f5c30effd8c.png)
![Untitled 6](https://user-images.githubusercontent.com/55252902/163293899-414db2d9-2afa-42cd-9125-93155eefab42.png)
![Untitled 7](https://user-images.githubusercontent.com/55252902/163293971-bfa5bf17-14db-4024-a97a-3b7918cc4ca3.png)


Ran linpeas on this system and foraging through all of that I found this...

![Untitled 8](https://user-images.githubusercontent.com/55252902/163293980-f049dcf0-af3b-4060-ada6-68cf7a8b33cb.png)

User vagrant is in root? I can’t switch to vagrant from this terminal, I need a interactive shell. 

![Untitled 9](https://user-images.githubusercontent.com/55252902/163294080-6f9a61c9-b2c9-470a-ac95-41a2b88e230e.png)


[Interact Shell Guide](https://netsec.ws/?p=337) or just google how to get to an interactive shell. 

![Untitled 10](https://user-images.githubusercontent.com/55252902/163294090-3b1258bd-6db9-42d3-9fd8-f2fd1f463f04.png)

Apparently vagrant’s password is the same was the user. 

![Untitled 11](https://user-images.githubusercontent.com/55252902/163294116-69f7fdba-6159-4baa-a6d5-60b6add838e2.png)
![Untitled 12](https://user-images.githubusercontent.com/55252902/163294123-9368a9e3-1ed7-494d-b949-7bc9e26eab4f.png)

Flag is base64 encoded and I can’t decode as vagrant, so I have to switch to james.

![Untitled 13](https://user-images.githubusercontent.com/55252902/163294149-32939e6f-5543-4acf-b421-52a3ca3139df.png)

Same thing can be done for root

![Untitled 14](https://user-images.githubusercontent.com/55252902/163294214-f8c7ae79-b4e2-4391-8dbf-90cdc1114277.png)

# Conclusion

More or less... This was an easy machine. What it taught me was WordPress. I had no experience with WordPress and it’s plugins. This machine made me do some research on WordPress and vulnerabilities associated with this version being run. I came across a very helpful tool, wpscan and going through a cheatsheet of wpscan I became familiar with it. 

### Sources 

[Paket storm](https://packetstormsecurity.com/files/163012/WordPress-wpDiscuz-7.0.4-Remote-Code-Execution.html)

[NIST CVE-2020-24186 Details](https://nvd.nist.gov/vuln/detail/CVE-2020-24186)

[CVE-2020-24186 Mitre](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-24186)

[Critical Arbitrary File Upload Vulnerability Patched in wpDiscuz Plugin](https://www.wordfence.com/blog/2020/07/critical-arbitrary-file-upload-vulnerability-patched-in-wpdiscuz-plugin/)









