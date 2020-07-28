---
title: "Fish Shell Setup"
date: 2020-07-28T10:18:14+12:00
tags:
  - Linux
---

[Fish shell](https://fishshell.com/) is an alternative to bash with a lot of quality of life improvements.
This post explains how I set up and configure Fish.
<!--more-->
The language is slightly different to bash, for example:

| Action                       | Bash                                | Fish                            |
|------------------------------|-------------------------------------|---------------------------------|
| Command substitution         | `"$(command)"`                      | `(command)`                     |
| Variable assignment          | `foo=bar`                           | `set foo bar`                   |
| Logical AND between commands | `command1 && command2`              | `command1; and command2`        |
| Logical NOT                  | `! command`                         | `not command`                   |
| if                           | `if CONDITION; then COMMAND; fi`    | `if CONDITION; COMMAND; end`    |
| for                          | `for VAR in LIST; do COMMAND; done` | `for VAR in LIST; COMMAND; end` |

You can find more examples in [this GitHub issue](https://github.com/fish-shell/fish-shell/issues/2382)

## Fish Installation
Now we can start Ubuntu, automatically logged in as our new user, from the Start menu. Go ahead and do that now.
In this process we'll be installing Fish, and the package manager [Oh My Fish](https://github.com/oh-my-fish/oh-my-fish)
```bash
# Start by installing Fish
sudo apt-add-repository ppa:fish-shell/release-3
sudo apt update
sudo apt -y install fish
sudo apt -y upgrade && sudo apt -y autoremove

# Next up we want to install Oh My Fish
curl -L https://get.oh-my.fish | fish

#Now that the Fish shell is installed, we can update our WSL config to use Fish:
chsh -s $(which fish)
logout
```

Next time you start Ubuntu, you should be immediately dropped into the Fish shell!
If you want, you can install a new theme. Check them out [here](https://github.com/oh-my-fish/oh-my-fish/blob/master/docs/Themes.md)
```fish
omf install bobthefish
```

This particular theme uses [Powerline](https://github.com/powerline/powerline), which the default Windows fonts don't support.
You'll need to install a [Powerline patched font](https://github.com/powerline/fonts), and then change the terminal properties to use this font.
I'm using the TTF with Powerline version of Microsoft's [Cascadia Code](https://github.com/microsoft/cascadia-code/releases) font (`CascadiaCodePL.ttf`).

## SSH Agent Setup
I use a couple of different SSH keys to remote in to various systems. It's handy if ssh-agent is started when you first open Ubuntu, and persists with your Windows session.

Generate a private key if you don't have one. Be sure to set a password. Functionally you don't have to, but it's good security practice.
```bash
ssh-keygen -t ed25519 -a 100
# Explanation of options:
#   -t ed25519   --> use the modern Ed25519 signature scheme
#   -a 100   --> run 100 rounds of key derivations, to make the key more brute-force resistant
```

Add any other private keys you may have to `~/.ssh` as `id_*` (for example `id_rsa-sean`, or `id_ed25519-work`), then set permissions correctly:
```bash
chmod 600 ~/.ssh/id_*
```

The following file will start ssh-agent when you first log in, and will persist the settings across multiple terminal sessions.
If you log out of Windows, restart, or otherwise kill `ssh-agent`, you'll be prompted for your key passwords when you next open Ubuntu.
Create the file `~/.config/fish/conf.d/ssh-agent.fish` with your favourite editor. I prefer `vim`.
```fish
set SSH_ENV "$HOME/.ssh/environment"

function addsshkeys
  set added_keys (ssh-add -l)
  for key in (find ~/.ssh/ -not -name "*.pub" -a -iname "id_*")
    if test ! (echo $added_keys | grep -o -e $key)
      ssh-add "$key"
    end
  end
end

function start_agent
    echo "Initialising new SSH agent..."
    /usr/bin/ssh-agent -c | sed 's/^echo/#echo/' > $SSH_ENV
    echo succeeded
    chmod 600 "$SSH_ENV"
    . "$SSH_ENV" > /dev/null
    addsshkeys
end

# Source SSH settings, if they exist
if status --is-interactive
    if test -f "$SSH_ENV"
        . "$SSH_ENV" > /dev/null
        # Check if agent is still running, if not, start a new one
        ps -ef | grep $SSH_AGENT_PID | grep "ssh-agent -c\$" > /dev/null; or start_agent;
    else
        echo "Environment file doesn't exist"
        start_agent
    end
end
```

## Install [NVM](https://github.com/nvm-sh/nvm) (Node.js Version Manager)
Only install this if you know that you need to use Node.js for something. NVM doesn't support Fish out of the box, but you can make it work as follows:
```bash
# Install NVM (version may have changed): https://github.com/nvm-sh/nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash

# Install the Fish NVM plugin: https://github.com/derekstavis/plugin-nvm
omf install nvm
```

## Install some more useful tools
### [homebrew](https://brew.sh/) - A third party package manager
```bash
curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh | bash
omf install linuxbrew

# Add Fish autocompletions
echo '
if test -d (brew --prefix)"/share/fish/completions"
    set -gx fish_complete_path $fish_complete_path (brew --prefix)/share/fish/completions
end
if test -d (brew --prefix)"/share/fish/vendor_completions.d"
    set -gx fish_complete_path $fish_complete_path (brew --prefix)/share/fish/vendor_completions.d
end
' >> ~/.config/fish/config.fish
```

### [exa](https://github.com/ogham/exa) - Improved `ls`
```bash
brew install exa
```

### [bat](https://github.com/sharkdp/bat) - More powerful version of `cat`
```bash
brew install bat
```

### [ncdu](https://dev.yorhel.nl/ncdu) - Interactive version of `du`, more like `windirstat`
```bash
sudo apt install ncdu
```

### [delta](https://github.com/dandavison/delta) - Like `diff` but better
```bash
brew install git-delta
```

### [GRC](https://github.com/garabik/grc) - Generic Colourizer to make your shell prettier
```bash
# Install the GRC package
sudo apt install grc

# Add automatic aliases for GRC, override ls alias to ensure it always shows its own colours
echo '
source /etc/grc.fish
function ls --inherit-variable executable --wraps=ls
    if isatty 1
        grc ls --color -C $argv
    else
        eval command ls $argv
    end
end
' > ~/.config/fish/conf.d/grc.fish
```

### Colourise `less`
The `highlight` application may slow down `less` too much for you, if so just remove the last `set` line.
```bash
sudo apt install highlight
echo '
set -xU LESS_TERMCAP_md (printf "\e[01;31m")
set -xU LESS_TERMCAP_me (printf "\e[0m")
set -xU LESS_TERMCAP_se (printf "\e[0m")
set -xU LESS_TERMCAP_so (printf "\e[01;44;33m")
set -xU LESS_TERMCAP_ue (printf "\e[0m")
set -xU LESS_TERMCAP_us (printf "\e[01;32m")In
set -xU LESS "--RAW-CONTROL-CHARS"
set -xU LESSOPEN "| /usr/bin/highlight %s --out-format xterm256 --force"
' > ~/.config/fish/conf.d/less.fish
```


## Further Fish config
Install the foreign environment plugin, to easily import your Bash profile. Only install this if you really need it.
```bash
omf install foreign-env
```
Edit `~/.config/fish/config.fish` to use the plugin to import your Bash profile on login
```bash
fenv source ~/.profile
```
