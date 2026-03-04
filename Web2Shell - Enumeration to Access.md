Nmap scan result
```shell
nmap -sV -sC 192.168.5.191     
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-07 11:28 WAT
Nmap scan report for 192.168.5.191
Host is up (0.45s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d8:e0:99:8c:76:f1:86:a3:ce:09:c8:19:a4:1d:c7:e1 (DSA)
|   2048 82:b0:20:bc:04:ea:3f:c2:cf:73:c3:d4:fa:b5:4b:47 (RSA)
|   256 03:4d:b0:70:4d:cf:5a:4a:87:c3:a5:ee:84:cc:aa:cc (ECDSA)
|_  256 64:cd:d0:af:6e:0d:20:13:01:96:3b:8d:16:3a:d6:1b (ED25519)
80/tcp open  http    Apache httpd 2.4.10 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.10 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.73 seconds

```

Enumerating the web address using dirsearch

![](attachments/Pasted%20image%2020260207115447.png)

Found an endpoint that pops a 403 error

![](attachments/Pasted%20image%2020260207115254.png)

> Let's try to bypass the 403 using 403bypass

![](attachments/Pasted%20image%2020260207121031.png)
found a payload encode upon usage....

Analyzing request n response in burp before attempted bypass
![](attachments/Pasted%20image%2020260207120507.png)

After applying encoded payload
![](attachments/Pasted%20image%2020260207121122.png)
nothing interesting

![](attachments/Pasted%20image%2020260207122407.png)

Lets try the 'x-fowarded-for' header

![](attachments/Pasted%20image%2020260207122611.png)
bypassed succesfully