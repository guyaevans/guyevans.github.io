---
title: Hugo, Caddy and Github actions to publish my website
date: 2024-10-11T15:28:00+02:00
rssFullText: true
lightgallery: true
featuredImage: feature.jpg
tags:
    - website
    - caddy
    - hugo
    - github
summary: I've always had some form of presence on the internet from a young age, I remember registering my own domain name aged 13 and building my first website with [Microsoft Frontpage](https://fr.wikipedia.org/wiki/Microsoft_FrontPage) and then later on with [Adobe Dreamweaver](https://en.wikipedia.org/wiki/Adobe_Dreamweaver). I later moved on to starting a career in IT and designing websites some websites with [Wordpress.](https://wordpress.org/). Let's take a look at what I use to publish this website today.
---
## Intro
I've always had some form of presence on the internet from a young age, I remember registering my own domain name aged 13 and building my first website with [Microsoft Frontpage](https://fr.wikipedia.org/wiki/Microsoft_FrontPage) and then later on with [Adobe Dreamweaver](https://en.wikipedia.org/wiki/Adobe_Dreamweaver). I later moved on to starting a career in IT and designing websites some websites with [Wordpress.](https://wordpress.org/).

These days I have moved away from Wordpress even though it is a great platform for posting a blog and building websites. (Wordpress is said to power 43 % of websites online)[^w3] but my website has evolved from just a mere placeholder to somewhere where I regularly post thoughts and content that I find interesting and hope that someone else finds interesting (At least I try ...).

I no longer want to faff around with Wordpress, Php, Mysql, the endless theme and plugins updates. And the constant hand-holding that it needs. I just want to concentrate on writing, and this is how the tech stack for my website has come to life. 
[^w3]: Report by [W3 Techs](https://w3techs.com/technologies/overview/content_management)

So what am I using to build and publish this website? Grab a drink and let's dive in.

{{<image src="/img/bbt-penny-drinkswebp.webp">}}

## Tech Stack
I've based it on three things at its core:

* [Hugo](https://gohugo.io/) - An open-source static site generator
* [Caddy](https://caddyserver.com/) - The actual web server to host the site
* [Github Actions](https://github.com/features/actions) - A service to build the static files with hugo and push them to the server running Caddy

Lets take a look at each service.
### Hugo

If you look around there are plenty of static site generators around, like [Ghost](https://ghost.org/), [Astro](https://astro.build/), [Jeykll](https://jekyllrb.com/) and [a boot load more](https://jamstack.org/generators) each using a different programming language. I can't quite remember how I found Hugo but it did stand out for its simplicity.

[Hugo](https://gohugo.io/) is an open-source site generator. Yes you read that right, I'm no web developer so no more PHP here. 

You feed it a theme, content written in [Markdown](https://www.markdownguide.org/) (bonus point) and HTML if needed. It also supports emojis :smile:, embedded videos, twitter posts (before Mister Elon killed it). Once your content is ready, Hugo converts everything into static HTML in seconds ready to upload to your web server.

[Hugo](https://gohugo.io/) also makes local development easy, by allowing you to spin up a local copy of your  website and get live feedback on your edits.

To get started all you need to do is [install it.](https://gohugo.io/getting-started/quick-start/)

### Caddy

[Caddy](https://caddyserver.com/), my love ... Where to start? I never got along the the configs for Apache or Nginx but Caddy's is simple.

[Caddy](https://caddyserver.com/) is a simple, fast and cross platform web server written in Go. I've [written about caddy here](https://guy-evans.com/posts/2023-04-23_a-look-at-caddy-a-simple-and-fast-web-server/) in the past when it was only serving parts of this website namely the comment system and analytics. But recently I moved the core of my website back to caddy because it's so simple.

Here the config for just the website (no comments or analytics)

```json {title=Caddyfile}
guy-evans.com www.guy-evans.com {
	root * /srv/guy-evans.com
	encode zstd gzip
	file_server
        log {
                output file /log/guy-evans-fr-caddy_access.log
  }
}
```
Yep, that's right. Eight lines of config to host the site 😎

### Github Actions

[Github Actions](https://github.com/features/actions) is a continuous integration and continuous delivery (CI/CD) platform that allows you to automate your build, test, and deployment pipeline. 
Basically speaking this allows me to automatically run ```hugo``` to build the static files and then upload them to my Caddy web server all from simple commit to the Github repo where I'm storing source code. 
No more manually building the static files locally on my laptop and then using SFTP to upload them to the server, it is all done for me.

```yaml {open=true}
name: Hugo CI & Deploy
on:
    push:
        branches:
            - master
    pull_request:
    # Allows you to run this workflow manually from the Actions tab
    workflow_dispatch:
jobs:
    build:
        name: Build and deploy website
        runs-on: ubuntu-latest
        steps:
        # We checkout the master branch with any submodules
            - name: Checkout
              uses: actions/checkout@v3
              with:
                  submodules: true
        # We setup Hugo with the version we want
            - name: Setup Hugo
              uses: peaceiris/actions-hugo@v2
              with:
                  hugo-version: "latest"
                  extended: true
        # We ask Hugo to build the website static files
            - name: Build website with Hugo
              run: hugo --minify
        # If this is a commit to the master branch we then use Rsync to send the files to the web server using the secrets to connect
            - name: Deploy website with rsync
              if: github.ref == 'refs/heads/master'
              uses: burnett01/rsync-deployments@5.2
              with:
                  switches: -avzr --quiet --delete
                  path: public/
                  remote_path: ${{ secrets.REMOTE_PATH }}
                  remote_host: ${{ secrets.REMOTE_HOST }}
                  remote_user: ${{ secrets.REMOTE_USER }}
                  remote_key: ${{ secrets.REMOTE_KEY }}
                  remote_port: ${{ secrets.REMOTE_PORT }}
```

Every time I make a commit to the master branch of my repo the workflow is run

## Wrap Up
As you see with this tech stack I can concentrate on writing content and no worry about the tech behind my website. I now have a simple and fast workflow to publish content. No PHP, no MYSQL, No Plugins to worry about. As usual everything is better with the KISS[^kiss] philosophy

[^kiss]: Keep it simple, stupid

--
_Photo by Negative Space at [Pexels](https://www.pexels.com/photo/gray-laptop-computer-showing-html-codes-in-shallow-focus-photography-160107/)_