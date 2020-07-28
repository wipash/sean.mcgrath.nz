---
title: "How To Build This Blog"
date: 2019-01-05T18:08:35+13:00
tags:
  - Hugo
---

This blog is built using [Hugo](https://gohugo.io/) and the [Beautiful Hugo](https://github.com/halogenica/beautifulhugo) theme.
The generated static pages are hosted using [Netlify](https://www.netlify.com/).
<!--more-->
### Software Assumptions
- Ubuntu
- [Fish](https://fishshell.com/) shell and [Homebrew](https://brew.sh/), set up with [this guide]({{< ref "/post/Fish-Shell-Setup" >}})

## Environment setup

First up, we need to install hugo.
```fish
brew install hugo
```

This site will live in a GitHub repo, so go ahead and create that. In this document, the repo is called `sean.mcgrath.nz`
Once the repo is created, clone it and create a new hugo site. Because the directory we're going to use will already exist (and may have files in it), we'll need to call `hugo` with the `--force` flag.
```fish
cd ~/dev
git clone git@github.com:wipash/sean.mcgrath.nz.git
cd sean.mcgrath.nz
hugo new site ./ --force
```

To ensure that git includes all the directories that have just been created (it will exclude empty dirs by default), create a .gitkeep file within each one.
This syntax is specific to [fish](https://fishshell.com/), but can easily be converted to bash.
```fish
for DIR in (ls -p | grep /); touch $DIR.gitkeep; end
```

We don't want the locally rendered site to ever be added to the repo, so exclude it by using the .gitignore file
```fish
echo "public" >> .gitignore
```

## Site setup

### Site configuration

Config for the site is controlled by `config.toml` in the root directory.
The settings I've set for this site are as follows:
```toml
baseURL = "https://sean.mcgrath.nz/"
languageCode = "en-us"
title = "Sean McGrath"
theme = "yinyang"                       # Name of the theme to use, matches the name of the
                                        #   folder in /themes/
newContentEditor = "vim"                # This is the editor that hugo will open when you create a new
                                        #   page or post
metaDataFormat = "toml"
PygmentsCodeFences = true
PygmentsCodefencesGuessSyntax = true
PygmentsStyle = "dracula"               # Code highlighting style
enableEmoji = true                      # Lets you emojify text like "\:heart\:" -> :heart:

[params]
  # disqus = "seanmcgrathnz"
  mainSections = ["post"]
  copyrightContent = "© 2019 Sean McGrath"
  extraHead = "<script async src='https://www.googletagmanager.com/gtag/js?id=UA-10214740-7'></script><script>window.dataLayer = window.dataLayer || []; function gtag(){dataLayer.push(arguments);} gtag('js', new Date()); gtag('config', 'UA-10214740-7');</script>"

[author]
  name = "Sean McGrath"
  homepage = "https://sean.mcgrath.nz"
  email = "sean@mcgrath.nz"
  github = "wipash"
  gitlab = "wipash"
  linkedin = "sean-m-mcgrath"

[[params.socials]]                      # Items to put in the bottom menu. These are double bracketed so
  name = "Github"                       #   that they become an array of tables. See the TOML spec:
  link = "https://github.com/wipash"    #   https://github.com/toml-lang/toml#array-of-tables
[[params.socials]]
  name = "GitLab"
  link = "https://gitlab.com/wipash"

```

### Theme

To ensure Netlify [properly loads the Hugo theme](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/#use-hugo-themes-with-netlify), the theme has to be added as a git [submodule](https://blog.github.com/2016-02-01-working-with-submodules/).
I picked [YinYang](https://github.com/joway/hugo-theme-yinyang) theme from the [Hugo Themes](https://themes.gohugo.io/) page, and [forked it](https://github.com/wipash/hugo-theme-yinyang) so that I could modify it a bit.
```
git submodule add git@github.com:wipash/hugo-theme-yinyang.git themes/yinyang
```
I haven't used them all of this theme's in the above config, but more can be found [here](https://github.com/halogenica/beautifulhugo/blob/master/exampleSite/config.toml)

### Netlify Configuration

Netlify gives the option to modify some basic settings when you connect it to your repo. To do more advanced configuration, create a file called `netlify.toml`
The full breakdown of options is available [here](https://www.netlify.com/docs/netlify-toml-reference/)
```toml
[build]
publish = "public/"                     # The Hugo output folder that Netlify will publish
command = "hugo --gc --minify"          # Run hugo with options to clean up unused cache files and
                                        #   minify output
[context.production.environment]        # Environment variables which Hugo will interpret
HUGO_VERSION = "0.74.3"
HUGO_ENV = "production"
HUGO_ENABLEGITINFO = "true"             # Lets the site access last Git revision information for
                                        #   every content file
```
