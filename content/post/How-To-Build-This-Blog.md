---
title: "How To Build This Blog"
date: 2019-01-05T18:08:35+13:00
---

This blog is built using [Hugo](https://gohugo.io/) and the [Beautiful Hugo](https://github.com/halogenica/beautifulhugo) theme.
The generated static pages are hosted using [Netlify](https://www.netlify.com/).
<!--more-->
### Software Assumptions
- Ubuntu
- [Fish](https://fishshell.com/) shell

## Environment setup

The commands here are somewhat specific to [fish](https://fishshell.com/), but can easily be converted to bash.
First up, we need to install hugo:
```fish
cd ~
set ver 0.53    # This is the fish way of setting variables
wget https://github.com/gohugoio/hugo/releases/download/v"$ver"/hugo_"$ver"_Linux-64bit.deb
sudo dpkg --install hugo_"$ver"_Linux-64bit.deb"
```

Beacuse we're using fish, we should get the autocompletion working for hugo
```
mkdir --parents ~/.config/fish/completions
wget https://raw.githubusercontent.com/fish-shell/fish-shell/master/share/completions/hugo.fish \
    --output-document ~/.config/fish/completions/hugo.fish
```

This site will live in a GitHub repo, so go ahead and create that. In this document, the repo is called `sean.mcgrath.nz`
Once the repo is created, clone it and create a new hugo site. Because the directory we're going to use will already exist (and may have files in it), we'll need to call `hugo` with the `--force` flag.
```
cd ~/dev
git clone https://github.com/wipash/sean.mcgrath.nz.git
cd sean.mcgrath.nz
hugo new site ./ --force
```

To ensure that git includes all the directories that have just been created (it will exclude empty dirs by default), create a .gitkeep file within each one.
Again, this syntax is fish specific.
```fish
for DIR in (ls -p | grep /); touch $DIR.gitkeep; end
```

We don't want the locally rendered site to ever be added to the repo, so exclude it by using the .gitignore file
```
echo "public" >> .gitignore
```

## Site setup

### Site configuration

Config for the site is controlled by `config.toml` in the root directory.
The settings I've set for this site are as follows:
```toml
baseURL = "https://sean.mcgrath.nz/"
languageCode = "en-us"
title = "Sean's Blog"
theme = "beautifulhugo"                 # Name of the theme to use, matches the name of the
                                        #   folder in /themes/
newContentEditor = "vim"                # This is the editor that hugo will open when you create a new 
                                        #   page or post
PygmentsCodeFences = true               # Lets you use three backticks to signify a code block, 
                                        #   GitHub Code Fences style
PygmentsCodefencesGuessSyntax = true
PygmentsStyle = "dracula"               # Code highlighting style

# 
# The following section is specific to the beautifulhugo theme
#
[Params]
  subtitle = "A blog about technology"
  rss = true
  comments = true
  readingTime = true
  socialShare = true

[Author]                                # Details to put in the footer
  name = "Sean McGrath"
  website = "https://sean.mcgrath.nz"
  email = "sean@mcgrath.nz"
  github = "wipash"
  gitlab = "wipash"
  linkedin = "sean-m-mcgrath"

[[menu.main]]                           # Items to put in the top menu. These are double bracketed so
  name = "Blog"                         #   that they become an array of tables. See the TOML spec:
  url = ""                              #   https://github.com/toml-lang/toml#array-of-tables
  weight = 1

[[menu.main]]
  name = "About"
  url = "page/about/"
  weight = 2

```

### Theme

To ensure Netlify [properly loads the Hugo theme](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/#use-hugo-themes-with-netlify), the theme has to be added as a git [submodule](https://blog.github.com/2016-02-01-working-with-submodules/).
I picked the [Beautiful Hugo](https://github.com/halogenica/beautifulhugo) theme from the [Hugo Themes](https://themes.gohugo.io/) page.
```
cd themes
git submodule add https://github.com/halogenica/beautifulhugo.git beautifulhugo
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
HUGO_VERSION = "0.53"
HUGO_ENV = "production"
HUGO_ENABLEGITINFO = "true"             # Lets the site access last Git revision information for 
                                        #   every content file
```