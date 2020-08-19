# Mr Robot CTF

The Mr Robot boot2root was created to feel like a hack from the series Mr Robot.  In this we will be attacking a webserver to gain access to the machine as a normal user and then from there we will escalate our privileges.  Throughout these steps are 3 keys we will need to find.  You can find the Mr Robot machine on Vulnhub using the link below.

<a href="https://www.vulnhub.com/entry/mr-robot-1,151/">Mr Robot</a>

## Tools

- Nmap  #For portscanning
- Wfuzz #For directory bruteforcing and discovery
- Firefox #To connect to the website
- Hydra #Used for username enumeration and password bruteforcing
- Terminal #Used to do everything else

## Scanning

To start off, we perform an nmap scan on the target machine to see what ports are open.
The nmap command I used for this box is as follows

```bash
nmap -sC -sV -p- -O -oA mr_robot <ip address>
```

The arguments used are as follows

- -sC -> Used to automatically perform default scripts on known ports.
- -sV -> Attempt to get the version of the service on a port
- -p- -> Scan all ports.  These will be TCP ports.
- -O  -> Try to determine the operating system.
- -oA mr_robot -> Output results to all types called mr_robot.&lt;extension&gt;
- &lt;ip address&gt; -> The ip address of the target machine. In my case, it was 10.10.148.81.

This will take time to run so grab a coffee while you wait.

Once finished, the following results I got were

<pre>
# Nmap 7.80 scan initiated Tue Aug 18 09:43:31 2020 as: nmap -sC -sV -oA mrrrobot -p- -O 10.10.148.81
Nmap scan report for 10.10.148.81
Host is up (0.12s latency).
Not shown: 65532 filtered ports
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open   ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
Device type: general purpose|specialized|storage-misc|WAP|printer
Running (JUST GUESSING): Linux 3.X|4.X|2.6.X|2.4.X (91%), Crestron 2-Series (89%), HP embedded (89%), Asus embedded (88%)
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 cpe:/o:crestron:2_series cpe:/h:hp:p2000_g3 cpe:/o:linux:linux_kernel:2.6.22 cpe:/h:asus:rt-n56u cpe:/o:linux:linux_kernel:3.4 cpe:/o:linux:linux_kernel:2.4
Aggressive OS guesses: Linux 3.10 - 3.13 (91%), Linux 3.10 - 4.11 (90%), Linux 3.13 or 4.2 (90%), Linux 3.2 - 3.8 (90%), Linux 4.2 (90%), Linux 4.4 (90%), Crestron XPanel control system (89%), Linux 3.12 (89%), Linux 3.13 (89%), Linux 3.2 - 3.5 (89%)
No exact OS matches for host (test conditions non-ideal).

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Aug 18 10:27:14 2020 -- 1 IP address (1 host up) scanned in 2622.94 seconds
</pre>

We see that ports 80 and 443 are open and port 22 is closed