# Metasploitable2-Linux(FTP)
I will be attacking Metasploitble2-Linux focusing on FTP. My next writeups on Metasploitable will focus on other vulnearbabilities. 

The first thing I did was log into this machine with the credentials provided from the download page. 

The user and password are the same | msfconsole.


# Netdiscover Scan
I Ran a netdiscover scan to discover target machine's IP.
You can either run a netdiscover scan on your (Kali) attacker machine or the metasploitable machine just run ifconfig. 

![Netdiscover](https://user-images.githubusercontent.com/55252902/149642912-76a118a6-4c0f-4f06-b746-b7ae34ce796c.JPG)

# Nmap Scan (FTP)
![FTP](https://user-images.githubusercontent.com/55252902/149642993-49ca0427-80b7-4bcf-8f33-334c7a2aa1e0.JPG)

# FTP Enumeration 
This FTP server allows for anonymous login but that is not what I will be doing.

From the nmap scan we know the service and version number that is running on this FTP server | vsftpd 2.3.4.

From discovering the version number and service I conducted research on known exploits/vulnerabilities for vsftpd 2.3.4.

From googling I found a Backdoor Command Execution exploit. This exploit will be conducted through metasploit and done manually.

# Metasploit

![MS1](https://user-images.githubusercontent.com/55252902/149643360-694e5e8e-bb6d-486f-999b-d6d843fdbae9.JPG)
![MS2](https://user-images.githubusercontent.com/55252902/149643363-388773d2-6e40-4ab1-b9a6-2aeb06dfec47.JPG)
![MS3](https://user-images.githubusercontent.com/55252902/149643368-da7d3f10-2374-4be1-a57b-33b6e9108f95.JPG)
![MS4](https://user-images.githubusercontent.com/55252902/149643375-90d65684-47de-43da-87a9-3eabe1e634f7.JPG)
![MS5](https://user-images.githubusercontent.com/55252902/149643379-0b0e5f10-b8db-4462-bea1-a069e4dd9c6e.JPG)

# Manual Exploitation
I found a manual exploit through github 

[Github Link For Manual Exploit](https://github.com/ahervias77/vsftpd-2.3.4-exploit)

![manual1](https://user-images.githubusercontent.com/55252902/149643487-2f2c7a76-7252-41a3-97c1-5f2b4c3376e2.JPG)

Author of this manual exploit has a readme that I will follow.

![manual2](https://user-images.githubusercontent.com/55252902/149643490-71b63b81-cb9a-4a99-8c1b-ea40b6657aa6.JPG)
Clone the repo to my machine

![manual3](https://user-images.githubusercontent.com/55252902/149643495-46d015c6-348f-4872-bdbf-a1b976fddc14.JPG)
Run the exploit 

# Conclusion 
Exploitation of ftp on metasploitable2 was quite easy and simple. I got through all of this without getting stuck.
I will continue to discover more vulnerabilies on Metasploitable2, doing writeups, and improving my pentesting skills.










