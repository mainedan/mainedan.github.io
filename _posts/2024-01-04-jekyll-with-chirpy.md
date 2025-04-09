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

# Copy the _site files to your html folder if not hosting on github

## Building Site in CI

This site already works with GitHub actions, just push it up and check the actions Tab.

