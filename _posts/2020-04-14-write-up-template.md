---
description: >-
  Zweilosec's writeup on the xxx-difficulty xxx machine xxx from 
  https://hackthebox.eu
title: Hack The Box - Writeup Template                    # Add title of the machine here
date: 2020-04-14 08:00:00 -0600                           # Change the date to match completion date
categories: [Hack the Box, Templates]                     # Change Templates to Writeup
tags: [htb, hacking, hack the box, template, redteam]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

## Download me on GitHub

Feel free to download and use this writeup template for Hack the Box machines for your own writeups.  Please let me where you post them so I can check them out and see how you completed the machines!  If you have any contributions to my site, feel free to leave an issue and pull request!

Fork this on [Zweilosec's GitHub](https://github.com/zweilosec)!

## HTB - Machine_Name

## Overview

![Descriptive information card about this machine](<machine>-0-infocard.png)

Short description to include any strange things to be dealt with

## Useful Skills and Tools

### Useful thing 1

description with generic example

### Useful thing 2

description with generic example

## Enumeration

### Nmap scan

I started my enumeration with an nmap scan of `10.10.10.xxx`.  The options I regularly use are: 

| `Flag` | Purpose |
| :--- | :--- |
| `-p-` | A shortcut which tells nmap to scan all ports |
| `-vvv` | Gives very verbose output so I can see the results as they are found, and also includes some information not normally shown |
| `-sC` | Equivalent to `--script=default` and runs a collection of nmap enumeration scripts against the target |
| `-sV` | Does a service version scan |
| `-oA $name` | Saves all three formats \(standard, greppable, and XML\) of output with a filename of `$name` |

## Initial Foothold

## Road to User

### Further enumeration

### Finding user creds

### User.txt


## Path to Power \(Gaining Administrator Access\)

### Enumeration as user `username`

### Getting a shell

### Root.txt

Thanks to [`<box_creator>`](https://www.hackthebox.eu/home/users/profile/<profile_num>) for something interesting or useful about this machine.

If you have comments, issues, or other feedback, or have any other fun or useful tips or tricks to share, feel free to contact me on Github at [https://github.com/zweilosec](https://github.com/zweilosec) or in the comments below!

If you like this content and would like to see more, please consider buying me a coffee! <a href="https://www.buymeacoffee.com/zweilosec"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=zweilosec&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff"></a>