---
description: >-
  How to use Github Pages to host a website with Jekyll Chirpy theme
title: How to use GitHub Pages to host a website with Jekyll Chirpy theme                    # Add title of the machine here
date: 2022-07-19 08:00:00 -0600                           # Change the date to match completion date
categories: [Tips & Tricks]                     # Categories
tags: [github, blog, build]     # TAG names should always be lowercase; add relevant tags
show_image_post: true                                    # Change this to true
#image: /assets/img/                # Add image here for post preview image
---

## How to use GitHub Pages to host a website with Jekyll Chirpy theme

### The Chirpy theme

This great looking theme (that this website uses!) has many great features as can be seen below:

#### Features
* Localized Layout
* Dark/Light Theme Mode
* Pinned Posts
* Hierarchical Categories
* Last Modified Date for Posts
* Table of Contents
* Auto-generated Related Posts
* Syntax Highlighting
* Mathematical Expressions
* Mermaid Diagram & Flowchart
* Disqus/Utterances/Giscus Comments
* Search
* Atom Feeds
* Google Analytics
* GA Pageviews Reporting
* SEO & Performance Optimization

## Step 1 - Prerequisites

Before building the website, your development machine needs to be set up to host and run [Jekyll](https://jekyllrb.com/) and [Git](https://git-scm.com/).  I also personally use and recommend [Visual Studio Code](https://code.visualstudio.com/) to do your actual writing and publishing as it is very simple to use.  

This tutorial assumes you are using Ubuntu, or other Debian-based Linux distribution.  Instructions may vary slightly for other operating systems.

### Ruby

The following must be installed first:

* Ruby version 2.5.0 or higher, including all development headers (check your Ruby version using `ruby -v`)
```
sudo apt-get install ruby-full build-essential zlib1g-dev
```
* RubyGems (should be included with Ruby, check your Gems version using `gem -v`)
* GCC and Make (should be included in your linux distro, check versions using `gcc -v`,`g++ -v`, and `make -v`)

Next, add the requisite lines to your `.bashrc` file by running each command below:

```
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Jekyll

Next, install Jekyll and Bundler

```
gem install jekyll bundler
```

#### Blog Migrations

If you’re switching to Jekyll from another blogging system, Jekyll’s importers can help you with the move. To learn more about importing your site to Jekyll, visit the [jekyll-import](https://import.jekyllrb.com/docs/home/) docs site.

### Git

You will want to publish your updates to your website semi-automatically through Git rather than copying all changes to the GitHub website.  This will allow you to trigger the automatic build Actions built into GitHub Pages and Jekyll.

```
sudo apt install Git
```

You will also need to add your username and email to Git with the following commands:

```
git config --global user.name githubUsername
git config --global user.email your@email.address
```

### Visual Studio Code (optional)

First, install Visual Studio Code from the Snap store (included in Ubuntu).

```
sudo snap install --classic code
```

![vscode interface showing which extensions to install](../../assets/img/vscode-extensions.png)

After installation you will want to install the two extensions `Markdown All in One` and `GitHub Pull Requests and Issues`.  These will enable you to edit your document as Markdown with syntax highlighting, and publish your page to GitHub, respectively.  

To enable the GitHub integration you will need to click on the new GitHub icon that has appeared on the left-side bar, then click the button to sign in.  A browser window will open asking you to authorize the connection.  Sign in and allow the request.

## Step 2 - Forking Chirpy from GitHub

Create a new repository by forking the [chirpy-starter](https://github.com/cotes2020/chirpy-starter/generate) repository and naming it `<UserName>.github.io`, using your GitHub username in place of `<UserName>`.  Make sure the 

https://github.com/cotes2020/jekyll-theme-chirpy#documentation

## Step 3 - Git Clone your repository

Next you will need to clone your repository to your dev machine.  Change directories to the location you want your site's files to exist (I created a folder called git in my user's home folder), then type the following command.

```
git clone https://github.com/<your repository name>
```

This copies all of the files from GitHub to your local machine and sets them up for git to use.

## Step 4 - Configuration

TODO: finish from here!!

Update the variables of _config.yml as needed. Some of them are typical options:

url
avatar
timezone
lang

## .gitignore

Create a file called `.gitignore` in the root of your project to prevent Git from trying to sync these files.

```
_site/
.sass-cache/
.jekyll-cache/
.jekyll-metadata
# Ignore folders generated by Bundler
.bundle/
vendor/
```
Running Local Server
You may want to preview the site contents before publishing, so just run it by first:

Update/install Ruby:

brew install ruby
Update/install Jekyll:

gem install bundler jekyll
Change the working directory to the github repo, then run:

bundle install
Run the local server:

bundle exec jekyll s
After a while, the local service will be published at http://127.0.0.1:4000.

## Hosting a jekyll page locally 

For local testing before publishing. I am doing this on Ubuntu, your instructions may vary slightly.  For 

I meant to publish these instructions for myself when I first moved over to jekyll and Github Pages for hosting for posterity, but now I have to rebuild my development VM...and I have forgotten how to locally host my site again!

## Updating Jekyll

If you followed our setup recommendations and installed Bundler, run bundle update jekyll or simply bundle update and all your gems will update to the latest versions.

## Troubleshooting

### webrick error

I got the following error while trying to run `jekyll server` : 

> cannot load such file -- webrick (LoadError)

To fix this I did the following:

1. Updated `github-pages`, `jekyll`, and `jekyll-feed` gems by running `gem install github-pages`, `gem install jekyll`, and `gem install jekyll-feed`. I had to do this step as a simple `bundle update` wasn't installing the latest version.
2. Run `bundle update` and `gem update`
3. Finally run `jekyll serve`

## References

* https://jekyllrb.com/docs/installation/ubuntu/
* https://stackoverflow.com/questions/65989040/bundle-exec-jekyll-serve-cannot-load-such-file

If you have comments, issues, or other feedback, or have any other fun or useful tips or tricks to share, feel free to contact me on Github at [https://github.com/zweilosec](https://github.com/zweilosec) or in the comments below!

If you like this content and would like to see more, please consider buying me a coffee! <a href="https://www.buymeacoffee.com/zweilosec"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=zweilosec&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff"></a>