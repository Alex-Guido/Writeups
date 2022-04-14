# Jangow 01
This is a vulnerable machine from vulnhub. 

[Jangow01 from vulnhub](https://www.vulnhub.com/entry/jangow-101,754/)

### Host Discovery Scan 
![scan](https://user-images.githubusercontent.com/55252902/163291256-1fbfd756-3a88-4bc6-b084-58b70b21282c.png)

Target = 10.0.2.4

## Nmap Scan
![nampscan](https://user-images.githubusercontent.com/55252902/163291382-f8db1104-2fe2-463b-914c-2832bb2647c1.JPG)

## Enumeration and Exploitation 
Apache/2.4.18 (Ubuntu) Server at 10.0.2.4 Port 80 

This is the page I get. Scrolling through there is an enter email form, I throw some gibberish in there to check for input validation and there is.

![Untitled](https://user-images.githubusercontent.com/55252902/163292033-45a0ac92-f426-4c89-ac99-d982dea36c40.png)
![Untitled 1](https://user-images.githubusercontent.com/55252902/163291422-339758d0-608a-4b27-9351-4fbba298c91f.png)
![Untitled 2](https://user-images.githubusercontent.com/55252902/163291442-c5682456-7084-4d53-9aa6-9fd7be364ef1.png)

Using DirBuster to find directories I come across therse directories /site/wordpress/config.php 

![Untitled 3](https://user-images.githubusercontent.com/55252902/163291517-38a00c5d-6f7d-4e12-83ad-cb15900ca4e6.png)

desafio02 seems like a user, I take note of this. 
![Untitled 4](https://user-images.githubusercontent.com/55252902/163291580-bb4627ae-95db-4041-8e0c-6b4b474ef6dd.png)

Going back to the main page to the buscar page, just to test, I threw in a ls and it accepted my input. Knowing that my input was accepted, I am going to check what the current directory is.

![Untitled 5](https://user-images.githubusercontent.com/55252902/163291713-c8d2e01a-3201-4645-a1bd-e0a2b610399d.png)

Going to check what is in the directory before where I currently am, html.
![Untitled 6](https://user-images.githubusercontent.com/55252902/163291791-691702a5-e71a-4fc4-bdbb-b510c656fa7d.png)
![Untitled 7](https://user-images.githubusercontent.com/55252902/163291801-5fd2c4b0-88dc-46d0-b07f-b84c56868588.png)

.backup, cat out that file.

![Untitled 8](https://user-images.githubusercontent.com/55252902/163291834-45d96149-0474-452b-81d9-04ba2c2b977d.png)

 I get credentials for a user.
![Untitled 9](https://user-images.githubusercontent.com/55252902/163291928-9225d66f-6283-46fe-b58a-659842ff3ac4.png)
![Untitled 10](https://user-images.githubusercontent.com/55252902/163291951-a5c5d5e4-0573-43db-9106-b7d8e7017c58.png)
![Untitled 11](https://user-images.githubusercontent.com/55252902/163291959-7e94d02f-292d-4822-9715-82268f25e891.png)
![Untitled 12](https://user-images.githubusercontent.com/55252902/163291968-9ad1cd8f-bf2f-4d30-911b-9808b1e6b2a7.png)

Logged directly into the VM and peeped around. Found the user flag but not much else, checked to see what kernel version is being run.

I found [this exploit from exploit-db](https://www.exploit-db.com/exploits/45010) but I am having trouble trying to run it on the VM.

The VM is in Portuguese, so the keyboard format is in Portuguese too. Through the FTP server I put the exploit into the machine and I can’t run it because I can’t type out the command to. I can’t type ./exploit. I have a 10-keyless keyboard and I grabbed a full keyboard to see if that would work. I’ve looked up the Portuguese keyboard format but anything I try I can’t get a /.

![Untitled 13](https://user-images.githubusercontent.com/55252902/163292022-c7c395ce-0db1-4a65-a1e2-63fcb89323b1.png)

# Conclusion
At this point I am just pulling my hair out trying to find a way to change the keyboard format.

I’m pretty sure after I run the privilege escalation exploit against this machine the flag would probably be in the root directory. If that particular exploit didn’t work, I’m 100% sure I’d find an exploit that would work. Overall this box was quite easy, the hardest part was trying to change the keyboard format.







