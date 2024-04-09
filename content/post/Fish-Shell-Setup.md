---
title: "Fish Shell & WSL Setup"
date: 2020-11-23T10:45:14+13:00
tags:
  - WSL
  - Linux
categories:
  - Linux
---

[Fish shell](https://fishshell.com/) is an alternative to bash with a lot of quality of life improvements.
This post explains how I set up and configure Fish, with a bunch of general WSL settings thrown in as well.
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

Next time you start a new Ubuntu terminal session, you should be immediately dropped into the Fish shell!
If you want, you can install a new theme. Check them out [here](https://github.com/oh-my-fish/oh-my-fish/blob/master/docs/Themes.md)
```fish
omf install bobthefish
```

This particular theme uses [Powerline](https://github.com/powerline/powerline), which the default Windows fonts don't support.
You'll need to install a [Powerline patched font](https://github.com/powerline/fonts), and then change the terminal properties to use this font.
I'm using the TTF with Powerline version of Microsoft's [Cascadia Code](https://github.com/microsoft/cascadia-code/releases) font (`CascadiaCodePL.ttf`).

I've made a couple of changes to the theme's default settings to speed it up:
```fish
echo "
set -g theme_display_vagrant no
set -g theme_display_ruby no
" >> ~/.config/fish/config.fish
```
## SSH & Git Setup
This section assumes you have 1Password, with your SSH key saved to your vault and the 'Use the SSH agent' developer setting enabled.
Read more about that here: [https://developer.1password.com/docs/ssh/agent/](https://developer.1password.com/docs/ssh/agent/)

{{< notice tip >}}
If you don't use 1Password, and want to proxy the ssh-agent running from within Windows:
{{< detail "SSH Agent Setup - Windows Agent" >}}
## SSH Agent Setup - Windows Agent
Assuming you have some SSH keys already in circulation, this updated guide makes it easier to share the keys between WSL2 and Windows.

To start with, ensure that the Windows OpenSSH client is installed, and the agent is running:
```powershell
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
Get-Service -Name ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

Add keys to the agent:
```
ssh-add C:\Users\sean\.ssh\id_rsa-sean
```

Once ssh-agent is running, you can use [npiperelay](https://github.com/jstarks/npiperelay/) to create a socket in WSL2 that connects to the namedpipe from ssh-agent running in Windows:
1. Run `sudo apt-get install socat`
2. Install npiperelay:
```fish
wget -O "$HOME/npiperelay.zip" "https://github.com/jstarks/npiperelay/releases/latest/download/npiperelay_windows_amd64.zip"
unzip -j "$HOME/npiperelay.zip" "npiperelay.exe" -d "$HOME/.ssh/"
rm "$HOME/npiperelay.zip"
chmod +x "$HOME/.ssh/npiperelay.exe"
```
3. Add the following startup command to `~/.config/fish/conf.d/wsl-ssh-agent.fish`:
```fish
set -x SSH_AUTH_SOCK $HOME/.ssh/agent.sock
ss -a | grep -q $SSH_AUTH_SOCK
if [ $status != 0 ]
    rm -f $SSH_AUTH_SOCK
    setsid nohup socat UNIX-LISTEN:$SSH_AUTH_SOCK,fork EXEC:"$HOME/.ssh/npiperelay.exe -ei -s //./pipe/openssh-ssh-agent" >/dev/null 2>&1 & disown
end
```

If you need to differentiate between keys, for example to use different github accounts, you can identify the keys based on their public keys.
1. Create public keys from all keys in the agent:
```fish
cd ~/.ssh
ssh-add -L | gawk ' { print $0 > $3 ".pub" } '
chmod 600 ~/.ssh/id_*
```
2. Identify the key you want to use in `~/.ssh/config` by this public key. For example:
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/sean@mcgrath.net.nz.pub
    IdentitiesOnly yes
Host workgithub
    HostName github.com
    User git
    IdentityFile ~/.ssh/sean@mycompany.co.nz.pub
    IdentitiesOnly yes
```
{{< /detail >}}

If you don't want any interaction with Windows at all, and just want to automatically load your keys into an ssh-agent when WSL starts:
{{< detail "SSH Agent Setup - Linux Agent" >}}
## SSH Agent Setup - Linux Agent
I use a couple of different SSH keys to remote in to various systems. It's handy if ssh-agent is started when you first open Ubuntu, and persists with your Windows session.

Generate a private key if you don't have one. Be sure to set a password. Functionally you don't have to, but it's good security practice.
```fish
ssh-keygen -t ed25519 -a 100
# Explanation of options:
#   -t ed25519   --> use the modern Ed25519 signature scheme
#   -a 100   --> run 100 rounds of key derivations, to make the key more brute-force resistant
```

Add any other private keys you may have to `~/.ssh` as `id_*` (for example `id_rsa-sean`, or `id_ed25519-work`), then set permissions correctly:
```fish
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
{{< /detail >}}
{{< /notice >}}

Create a fish config file to alias `ssh` and `ssh-add` to the Windows executables:
```bash
echo '
function ssh-add
  /mnt/c/Windows/System32/OpenSSH/ssh-add.exe $argv
end

function ssh
  /mnt/c/Windows/System32/OpenSSH/ssh.exe $argv
end
' >> ~/.config/fish/conf.d/aliases.fish
```

Configure `git` to use Windows `ssh.exe`:
```bash
git config --global core.sshCommand ssh.exe
```

## Make WSL start faster
I've found that WSL can take a long time to start, if you have a lot of things in your Windows PATH variable. You can disable adding Windows
PATH variables to WSL by creating a `wsl.conf` file, see this reference: [https://learn.microsoft.com/en-us/windows/wsl/wsl-config#interop-settings](https://learn.microsoft.com/en-us/windows/wsl/wsl-config#interop-settings)

```bash
echo '
[interop]
appendWindowsPath = false
' >> /etc/wsl.conf
```

This has the side effect of breaking some useful WSL integrations, like running VS Code in Windows connected to WSL by typing `code`. You can fix that by adding another alias (you might need to update the path to VS Code if you have it installed in a different location):
```bash
echo '
function code
  /mnt/c/Program\ Files/Microsoft\ VS\ Code/bin/code $argv
end
' >> ~/.config/fish/conf.d/aliases.fish
```

## Install [NVM](https://github.com/nvm-sh/nvm) (Node.js Version Manager)
Only install this if you know that you need to use Node.js for something. NVM doesn't support Fish out of the box, but you can make it work as follows:
```fish
# Install NVM (version may have changed): https://github.com/nvm-sh/nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash

# Install the Fish NVM plugin: https://github.com/derekstavis/plugin-nvm
omf install nvm
```

## Install some more useful tools
### [homebrew](https://brew.sh/) - A third party package manager
```fish
curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh | bash

# Add Homebrew to path, and configure Fish autocompletions
echo 'set -l brew "/home/linuxbrew/.linuxbrew/bin/brew"

if [ -d "$HOME/.linuxbrew" ]
    set brew "$HOME/.linuxbrew/bin/brew"
end

if [ -f "$brew" ]
    eval (eval "$brew" shellenv)
end
if test -d (brew --prefix)"/share/fish/completions"
    set -gx fish_complete_path $fish_complete_path (brew --prefix)/share/fish/completions
end
if test -d (brew --prefix)"/share/fish/vendor_completions.d"
    set -gx fish_complete_path $fish_complete_path (brew --prefix)/share/fish/vendor_completions.d
end
' > ~/.config/fish/conf.d/homebrew.fish
```

### [exa](https://the.exa.website/) - Improved `ls`
```fish
brew install exa
```

### [bat](https://github.com/sharkdp/bat) - More powerful version of `cat`
```fish
brew install bat
```

### [ncdu](https://dev.yorhel.nl/ncdu) - Interactive version of `du`, more like `windirstat`
```fish
sudo apt install ncdu
```

### [delta](https://github.com/dandavison/delta) - Like `diff` but better
```fish
brew install git-delta
```

### [GRC](https://github.com/garabik/grc) - Generic Colourizer to make your shell prettier
```fish
# Install the GRC package
sudo apt install grc

# Add automatic aliases for GRC, override ls alias to ensure it always shows its own colours
echo 'source /etc/grc.fish
function ls --inherit-variable executable --wraps=ls
    if isatty 1
        grc ls --color -C -w(tput cols) $argv
    else
        eval command ls $argv
    end
end
' > ~/.config/fish/conf.d/grc.fish
```

### [neovim](https://github.com/neovim/neovim) - Better Vim
`neovim` is a continuation and extension of `vim`. I'm using the [`space-vim`](https://github.com/liuchengxu/space-vim) distribution which offers an improved interface and [key mappings](http://liuchengxu.org/space-vim-doc/tutorial/space-vim/). You can run `neovim` using the command `nvim`.
```fish
sudo apt install neovim
curl https://raw.githubusercontent.com/liuchengxu/space-vim/master/install.sh -o ~/spacevim.sh
chmod +x ~/spacevim.sh
./spacevim.sh --nvim
```

Add the [`vim-fish`](https://github.com/blankname/vim-fish) plugin under the `UserInit()` section in the space-vim config file. This plugin offers syntax hilighting for `.fish` config files.
```fish
nvim ~/.spacevim
# Insert the following line under function! UserInit():
Plug 'blankname/vim-fish'
```

### Colourise `less`
The `highlight` application may slow down `less` too much for you, if so just remove the last `set` line.
```fish
sudo apt install highlight
echo 'set -xU LESS_TERMCAP_md (printf "\e[01;31m")
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
```fish
omf install foreign-env
```
Edit `~/.config/fish/config.fish` to use the plugin to import your Bash profile on login
```fish
fenv source ~/.profile
```
