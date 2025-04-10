---
title: Jekyll with Chirpy Theme
author: Dan
date: 2024-01-04 10:00:00 -400
categories: [Website, Documentation]
tags: [homelab,documentation,website]
---

# Jekyll _Static Site Generator

This doc uses Techno Tim's guide https://technotim.live/posts/jekyll-docs-site/

## Install dependencies

```bash
sudo apt update
sudo apt install ruby-full build-essential zlib1g-dev git
```

To avoid installing RubyGems packages as the root user:

If you are using bash (usually the default for most)

```bash
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
Install Jekyll bundler

```bash
gem install jekyll bundler
```

# Creating a site based on Chirpy Starter

Visit https://github.com/cotes2020/jekyll-theme-chirpy#quick-start

After creating a site based on the template, clone your repo

```bash
git clone git@<YOUR-USER-NAME>/<YOUR-REPO-NAME>.git
```

Then install your dependencies

```bash
cd repo-name
bundle
```

After making changes to your site, commit and push then up to git

```bash
git add .
git commit -m "made some changes"
git push
```

# Jekyll Commands

Serving your site

```bash
bundle exec jekyll s
```

# Building your site in production mode

```bash
JEKYLL_ENV=production bundle exec jekyll b
```

## This will output the production site to _site

Copy the _site files to your html folder if not hosting on github

## Building Site in CI



This site already works with GitHub actions, just push it up and check the actions Tab., 

## Creating a Post 

### Naming Conventions 

Jekyll uses a naming convention for pages and posts 

Create a file in _posts with the format 

YEAR-MONTH-DAY-title.md 
 

For example: 

2022-05-23-homelab-docs.md 
2022-05-34-hardware-specs.md 
 

Jekyll can delay posts which have the date/time set for a point in the future determined by the “front matter” section at the top of your post file. Check the date & time as well as time zone if you don’t see a post appear shortly after re-build. 

## Local Linking of Files 

Image from asset: 

... which is shown in the screenshot below: 

![A screenshot](/assets/images/screenshot.png)

```bash
![A screenshot](/assets/screenshot.jpg) 
```

Linking to a file 

```bash
... you can [download the PDF](/assets/diagram.pdf)here. 
 ```

See more post formatting rules on the [Jekyll site](https://jekyllrb.com/) 

Markdown Examples 

If you need some help with markdown, check out the [markdown cheat sheet](https://www.markdownguide.org/cheat-sheet/) 

For more neat syntax for the Chirpy theme check their demo page on making posts https://chirpy.cotes.page/posts/write-a-new-post/ 
