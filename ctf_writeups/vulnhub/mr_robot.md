# Mr Robot CTF

The Mr Robot boot2root was created to feel like a hack from the series Mr Robot.  In this we will be attacking a webserver to gain access to the machine as a normal user and then from there we will escalate our privileges.  Throughout these steps are 3 keys we will need to find.  You can find the Mr Robot machine on Vulnhub using the link below.

<a href="https://www.vulnhub.com/entry/mr-robot-1,151/">Mr Robot</a>

## Tools

- **Nmap**  #For portscanning
- **Wfuzz** #For directory bruteforcing and discovery
- **WGet**  #To pull down files off the site.
- **Firefox** #To connect to the website
- **Hydra** #Used for username enumeration and password bruteforcing
- **Terminal** #Used to do everything else

## Wordlists

For this, I used the common.txt file found in Daniel Miessler's Github of collected wordlists.  You can find the file here

[common.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/common.txt)

While that wordlist was used for this box, I do recommend just pulling the full repository down.  You can do that via the following git command

```bash
git clone https://github.com/danielmiessler/SecLists.git
```

You should have a SecLists directory whereever you initiated the clone command.  The wordlists found in this repository are excellent for many different tasks.

## Scanning

To start off, we perform an nmap scan on the target machine to see what ports are open.
The nmap command I used for this box is as follows

```bash
nmap -sC -sV -p- -O -oA mr_robot <ip address>
```

The arguments used are as follows
<br/>

|Argument|Explanation|
|----|----|
**-sC** | Used to automatically perform default scripts on known ports.
**-sV** | Attempt to get the version of the service on a port
**-p-** |Scan all ports.  These will be TCP ports.
**-O**  | Try to determine the operating system.
**-oA** mr_robot | Output results to all types called mr_robot.&lt;extension&gt;
**&lt;ip address&gt;** | The ip address of the target machine. In my case, it was 10.10.148.81.

<br />
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

We see that ports 80 and 443 are open and port 22 is closed.  We also see the ports return header information about the server, Apache.  We can investigate this using a browser to see what the page looks like.

Open the browser and in the URL bar enter http://&lt;ip address&gt;.  This should take you to a page that looks like a console.

## Website 

Now that we know there is a web site on the target machine I typically do some quick manual investigations, including looking at the source code, clicking any links and buttons, and entering data into fields in order to get a feel for the site.  You can type some commands in the page and it will present some information to you but this is largely a red herring.  The other bit I look at manually are site.xml and robots.txt.  When you open the robots.txt file you'll see 2 files.  One with the first key and another called fsocity.dic.

<!-- Insert screenshot of robots.txt -->

We can use wget here to pull the fsocity.dic file down.  The command used is as follows

```bash
wget http://<ip address>/fsocity.dic
```

You should now have the fsocity.dic file.  We'll need this file later but first we need to figure out where.  That's where wfuzz will come in.

### WFuzz

We will just start a fuzz of the directories on the site using the following command

```bash
wfuzz -u http://<ip address>/FUZZ -w /path/to/common.txt --hc 404,403 -L --oF mr_robot_webscan_common
```

The arguments are as follows

|Argument|Explanation|
|----|----|
|**-u http://&lt;ip address&gt;/FUZZ** | The url of the web page.  In this case the /FUZZ are the directories we are going to be bruteforcing. |
|**-w /path/to/common.txt** | The wordlist used to replace the /FUZZ in the URL.|
|**--hc 404,403** | Hide any page that returns the status codes 404 and 403. |
|**-L**| Follow any redirects |
|**-oF mr_robot_webscan_common** | Output the results to mr_robot_webscan_common|

The results I got were numerous so I have cut out the less import results.

<pre>
********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.148.81/FUZZ
Total requests: 4658

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                                               
===================================================================

...Trimmed...

000004519:   200        0 L      0 W      0 Ch        "wp-config"                                                                                                                           
000004521:   200        0 L      0 W      0 Ch        "wp-cron"                                                                                                                             
000004520:   200        0 L      0 W      0 Ch        "wp-content"                                                                                                                          
000004513:   200        52 L     158 W    2664 Ch     "wp-admin"                                                                                                                            
000004527:   200        10 L     22 W     227 Ch      "wp-links-opml"                                                                                                                       
000004529:   200        52 L     158 W    2664 Ch     "wp-login"                                                                                                                            
000004528:   200        0 L      0 W      0 Ch        "wp-load"                                                                                                                             
000004536:   500        0 L      0 W      0 Ch        "wp-settings"                                                                                                                         
000004530:   500        109 L    300 W    3064 Ch     "wp-mail"                                                                                                                             
000004537:   200        54 L     169 W    2805 Ch     "wp-signup"
000004537:   200        54 L     169 W    2805 Ch     "wp-signup"                                                                                                                           
000004592:   405        0 L      6 W      42 Ch       "xmlrpc.php"                                                                                                                          
000004591:   405        0 L      6 W      42 Ch       "xmlrpc" 
</pre>

These results are important because it tells us this site is written using Wordpress.  Furthermore, we now have the login page we want to try logging into.

Navigate to **http://&lt;ip address&gt;/wp-login.php** and you will be presented with a login page (duh).

