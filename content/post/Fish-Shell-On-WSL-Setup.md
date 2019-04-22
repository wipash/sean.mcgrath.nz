+++
title = "Fish Shell on WSL/Ubuntu Setup"
date = 2019-02-06T17:02:19-05:00
+++
[Fish shell](https://fishshell.com/) is an alternative to bash with a lot of quality of life improvements.
This post explains how I set up Ubuntu on WSL, and configure Fish.
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

## Install WSL
Open up PowerShell and install the Windows Subsystem for Linux feature, then reboot:
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
Restart-Computer
```

Once rebooted, run the following PowerShell to download and install the latest (as of writing) LTS version of Ubuntu
```powershell
Invoke-WebRequest -Uri https://aka.ms/wsl-ubuntu-1804 -OutFile Ubuntu.appx -UseBasicParsing
Add-AppxPackage -Path ~/Ubuntu.appx
```
Alternatively, install your favourite distribution from the Microsoft Store.

## Initial WSL Config
I want to use a username that is forbidden by the `NAME_REGEX` in Ubuntu, so we'll initially configure Ubuntu with the `root` user and then create a new user once logged in.
```powershell
ubuntu install --root
ubuntu
```
Now, within the Ubuntu shell, create a new user:
```bash
# Add the new user
adduser sean.mcgrath --force-badname
# Add the user to the group allowed to run sudo
usermod -G sudo sean.mcgrath
# Set a password for the new user
passwd sean.mcgrath

# Make sure we're up to date
apt update && apt upgrade -y

logout
```
Back in PowerShell, update WSL's config to always log in with your new user
```powershell
ubuntu config --default_user sean.mcgrath
```

## Add Windows Defender exclusions for WSL
Windows Defender realtime protection has a significant performance impact to WSL.
Use the following PowerShell script (from an administrative PowerShell prompt) to exclude WSL from Windows Defender realtime scanning:

Credit: https://gist.github.com/noelbundick/9c804a710eb76e1d6a234b14abf42a52#file-excludewsl-ps1
```powershell
# Find registered WSL environments
$wslPaths = (Get-ChildItem HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss | ForEach-Object { Get-ItemProperty $_.PSPath}).BasePath

# Get the current Windows Defender exclusion paths
$currentExclusions = $(Get-MpPreference).ExclusionPath
if (!$currentExclusions) {
  $currentExclusions = ''
}

# Find the WSL paths that are not excluded
$exclusionsToAdd = ((Compare-Object $wslPaths $currentExclusions) | Where-Object SideIndicator -eq "<=").InputObject

# List of paths inside the Linux distro to exclude (https://github.com/Microsoft/WSL/issues/1932#issuecomment-407855346)
$dirs = @("\bin", "\sbin", "\usr\bin", "\usr\sbin", "\usr\local\bin", "\usr\local\go\bin")

# Add the missing entries to Windows Defender
if ($exclusionsToAdd.Length -gt 0) {
  $exclusionsToAdd | ForEach-Object {

    # Exclude paths from the root of the WSL install
    Add-MpPreference -ExclusionPath $_
    Write-Output "Added exclusion for $_"

    # Exclude processes contained inside WSL
    $rootfs = $_ + "\rootfs"
    $dirs | ForEach-Object {
        $exclusion = $rootfs + $_ + "\*"
        Add-MpPreference -ExclusionProcess $exclusion
        Write-Output "Added exclusion for $exclusion"
    }
  }
}
```

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
I'm using [DejaVe Sans Mono for Powerline](https://github.com/powerline/fonts/tree/master/DejaVuSansMono).

## SSH Agent Setup
I use a couple of different SSH keys to remote in to various systems. It's handy if ssh-agent is started when you first open Ubuntu, and persists with your Windows session.

Generate a private key if you don't have one. Be sure to set a password. Functionally you don't have to, but it's good security practice.
```bash
ssh-keygen -t rsa -b 4096 -o -a 100
# Explanation of options:
#   -t rsa   --> use RSA
#   -b 4096  --> generate a 4096 bit key
#   -o       --> use then OpenSSH key format, instead of the default PEM
#   -a 100   --> run 100 rounds of key derivations, to make the key more brute-force resistant

# Alternatively you can generate a key using the new Ed25519 scheme,
#  which may not have complete adoption yet: https://ianix.com/pub/ed25519-deployment.html
ssh-keygen -t ed25519 -a 100
```

Add your other private keys to `~/.ssh` as `id_rsa*` (for example `id_rsa-sean`), then set permissions correctly
```bash
chmod 600 ~/.ssh/id_rsa*
```

The following file will start ssh-agent when you first log in, and will persist the settings across multiple terminal sessions.
If you log out of Windows, restart, or otherwise kill `ssh-agent`, you'll be prompted for your key passwords when you next open Ubuntu.
Create the file `~/.config/fish/conf.d/ssh-agent.fish` with your favourite editor. I prefer `vim`.
```fish
set SSH_ENV "$HOME/.ssh/environment"

function addsshkeys
  set added_keys (ssh-add -l)
  for key in (find ~/.ssh/ -not -name "*.pub" -a -iname "id_rsa*")
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
if test -f "$SSH_ENV";
    . "$SSH_ENV" > /dev/null
    # Check if agent is still running, if not, start a new one
    ps -ef | grep $SSH_AGENT_PID | grep "ssh-agent -c\$" > /dev/null; or start_agent;
else
    echo "Environment file doesn't exist"
    start_agent
end
```