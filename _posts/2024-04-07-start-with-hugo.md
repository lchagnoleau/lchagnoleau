---
title: Setup a personal blog with Hugo
date: 2024-04-07
categories: [website]
tags: [hugo, github]
---

I decided to create a personal blog to share my thoughts, ideas, and projects with the world. I chose Hugo as my static site generator because of its simplicity, speed, and flexibility.
So, it's a good idea to start this blog by explaining how to setup a personal blog with Hugo and deploy it on GitHub Pages.

# Prerequisites

## Choose a appropriate theme

Hugo has a lot of themes available on the [official website](https://themes.gohugo.io/). You can choose one that fits your needs and customize it to your liking. For this blog, I chose the [Nightfall](https://themes.gohugo.io/themes/hugo-theme-nightfall/) theme because of its clean design and simplicity.

## Installation

To install Hugo, you can follow the instructions on the [official website](https://gohugo.io/getting-started/installing/). I'm using Arch Linux, so I installed Hugo with the following command:
```bash
sudo pacman -S hugo
```

Because I choose the Nightfall theme, I need to install [dart-sass](https://gohugo.io/functions/resources/tocss/#dart-sass), following the instructions in the [Nightfall repository](https://github.com/LordMathis/hugo-theme-nightfall/tree/main)
```bash
sudo pacman -S dart-sass
```

# Start a new site

To create a new site with Hugo, you can use the following command:
```bash
hugo new site personal_blog
```

This will create a new directory called `personal_blog`.
Next, we need to init hugo mod to be able to use the theme
```bash
hugo mod init github.com/lchagnoleau/personal_blog
hugo mod get -u
```

And add the following line to the `hugo.toml` file:
```toml
[module]
[[module.imports]]
  path = 'github.com/LordMathis/hugo-theme-nightfall'
```

To add a new post, you can use the following command:
```bash
hugo new content posts/my-first-post.m
```

Write your post in markdown format in the file `content/posts/my-first-post.md` and set the `draft` field to `false` to publish it.

Finally, we can start the server with the following command:
```bash
hugo server
```

# Publish on Github

## Create a Github repository

I push it into the .gitignore file
```bash
*.lock
public/
resources/
```

Then, create your Github repository and push your code to it
```bash
git init .
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:lchagnoleau/lchagnoleau.git
git push -u origin master
```

## Deploy on Github Pages

The instructions to deploy the blog on Github Pages are available [here](https://gohugo.io/hosting-and-deployment/hosting-on-github/).
