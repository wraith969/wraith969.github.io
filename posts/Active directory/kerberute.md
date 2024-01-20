First clone the repository
`git clone #https://github.com/ropnop/kerbrute.git`

cd kerbrute 

now lets modify the makefile 
sudo nano MakeFile
	add arm64 to the architecture
	ARCHS=amd64 386 arm64

install Golang 
	sudo apt install gccgo-go && apt install golang-go


now lets create the file 
	Make file 
	
┌──(wraith㉿kali)-[~/thm/tools/kerbrute]
└─$ make linux
Building for linux amd64...
go: downloading github.com/op/go-logging v0.0.0-20160315200505-970db520ece7
go: downloading github.com/ropnop/gokrb5/v8 v8.0.0-20201111231119-729746023c02
go: downloading github.com/spf13/cobra v1.1.1
go: downloading github.com/jcmturner/gofork v1.0.0
go: downloading github.com/spf13/pflag v1.0.5
go: downloading github.com/jcmturner/dnsutils/v2 v2.0.0
go: downloading github.com/hashicorp/go-uuid v1.0.2
go: downloading golang.org/x/crypto v0.0.0-20201016220609-9e8e0b390897
go: downloading github.com/jcmturner/rpc/v2 v2.0.2
go: downloading github.com/jcmturner/aescts/v2 v2.0.0
go: downloading golang.org/x/net v0.0.0-20200114155413-6afb5195e5aa
Building for linux 386...
Building for linux arm64...
Done.


CD DIST

┌──(wraith㉿kali)-[~/thm/tools/kerbrute/dist]
└─$ ls -la     
total 22548
drwxr-xr-x 2 wraith wraith    4096 Jan 14 04:55 .
drwxr-xr-x 8 wraith wraith    4096 Jan 14 04:55 ..
-rwxr-xr-x 1 wraith wraith 7399279 Jan 14 04:55 kerbrute_linux_386
-rwxr-xr-x 1 wraith wraith 7945397 Jan 14 04:55 kerbrute_linux_amd64
-rwxr-xr-x 1 wraith wraith 7725705 Jan 14 04:55 kerbrute_linux_arm64



./kerbrute_linux_arm64

┌──(wraith㉿kali)-[~/thm/tools/kerbrute/dist]
└─$ ./kerbrute_linux_arm64 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 01/14/24 - Ronnie Flathers @ropnop

This tool is designed to assist in quickly bruteforcing valid Active Directory accounts through Kerberos Pre-Authentication.
It is designed to be used on an internal Windows domain with access to one of the Domain Controllers.
Warning: failed Kerberos Pre-Auth counts as a failed login and WILL lock out accounts

Usage:
  kerbrute [command]

Available Commands:
  bruteforce    Bruteforce username:password combos, from a file or stdin
  bruteuser     Bruteforce a single user's password from a wordlist
  help          Help about any command
  passwordspray Test a single password against a list of users
  userenum      Enumerate valid domain usernames via Kerberos
  version       Display version info and quit

Flags:
      --dc string          The location of the Domain Controller (KDC) to target. If blank, will lookup via DNS
      --delay int          Delay in millisecond between each attempt. Will always use single thread if set
  -d, --domain string      The full domain to use (e.g. contoso.com)
      --downgrade          Force downgraded encryption type (arcfour-hmac-md5)
      --hash-file string   File to save AS-REP hashes to (if any captured), otherwise just logged
  -h, --help               help for kerbrute
  -o, --output string      File to write logs to. Optional.
      --safe               Safe mode. Will abort if any user comes back as locked out. Default: FALSE
  -t, --threads int        Threads to use (default 10)
  -v, --verbose            Log failures and errors

Use "kerbrute [command] --help" for more information about a command.
                                                                                                                               





