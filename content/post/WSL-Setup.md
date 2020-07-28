---
title: "WSL Setup"
date: 2020-07-28T10:17:25+12:00
tags:
  - WSL
  - Linux
---

This post explains how to install WSL2, and install and configure Ubuntu.
<!--more-->

## Install WSL
Open up PowerShell as Administrator and install the Windows Subsystem for Linux feature, then reboot:
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
Restart-Computer
```

Once rebooted, open PowerShell as Administrator again and run the following to configure WSL2 as the default. You can read about the differences between WSL1 and WSL2 [here](https://docs.microsoft.com/en-us/windows/wsl/compare-versions).
```powershell
wsl --set-default-version 2
```

Now, install your favourite distribution from the Microsoft Store. I recommend [Ubuntu](https://www.microsoft.com/store/productId/9NBLGGH4MSV6).

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
do-release-upgrade

logout
```
Back in PowerShell, update WSL's config to always log in with your new user
```powershell
ubuntu config --default_user sean.mcgrath
```

## Windows Terminal
For the best WSL experience, I recommend using [Windows Terminal](https://www.microsoft.com/store/productId/9N0DX20HK701), which you can also find in the Microsoft Store.

Because the theme I use with Fish shell includes special characters for Powerline I have installed the TTF with Powerline version of Microsoft's [Cascadia Code](https://github.com/microsoft/cascadia-code/releases) font (`CascadiaCodePL.ttf`). To set this font as the default for Windows Terminal, hit `Ctrl`+`,` to open the settings file and change the configuration like this:

```json
{
  "profiles": {
    "defaults": {
      "fontFace": "Cascadia Code PL"
    }
  }
}
```