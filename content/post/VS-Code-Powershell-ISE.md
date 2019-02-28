---
title: "VS Code as a Replacement for PowerShell ISE"
date: 2019-02-28T11:44:42-05:00
---

Microsoft has confirmed that the current PowerShell 5.1 will be the last Windows-specific version. With this announcement comes confirmation that PowerShell ISE will not be updated to support PowerShell 6 (aka **PowerShell Core**, you'll see it named one or the other in different places).

If you want to move to PowerShell 6 and still maintain a nice development environment like ISE, your best bet is to move to [Visual Studio Code](https://code.visualstudio.com/).

This guide will help you set up PowerShell 6, VS Code, and configure VS Code to act a bit like ISE.

Install PowerShell 6
-----------------------
PowerShell 6 is not yet ready to be a full replacement for PowerShell 5.1.
Therefore, the installer will install PowerShell 6 side-by-side with PowerShell 5.1, and will not conflict in any way.

Microsoft has [announced](https://techcommunity.microsoft.com/t5/PowerShell-AMA/Will-powershell-core-become-default/td-p/143824) that they're working on building PowerShell 6 into Windows, but no official timeline has been set yet.

1. Follow the steps [here](https://aka.ms/getps6-windows) to download the latest version of PowerShell 6 (or go directly to the [GitHub releases page](https://github.com/PowerShell/PowerShell/releases))
2. Run the installer and follow the prompts


Install PowerShell Modules
--------------------------
Because PowerShell 6 is a separate entity from PowerShell 5.1, you will need to install any modules that you often use.

For example, to install the [Azure PowerShell module](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-1.4.0#install-the-azure-powershell-module-1):

* Open PowerShell 6 as Administrator from the Start Menu
* Install the module: `Install-Module -Name Az -AllowClobber`


Install VS Code
---------------
1. Download the latest version of VS Code from [here](https://aka.ms/win32-x64-user-stable)
2. Run the installer and follow the prompts


Configure VS Code
-----------------
VS Code is extremely extensible through the use of **Extensions**.

To start installing extensions, open the Extensions sidebar, and then search for and install the **PowerShell** extension by Microsoft.

You're basically ready to go now, but the experience can be greatly improved by setting some configuration options.

1. Open **File -> Preferences -> Settings**, or press <kbd>Ctrl</kbd>+<kbd>,</kbd>
2. Click the <kbd>{}</kbd> button in the top right corner to **Open Settings (JSON)**
3. Add the following lines:

```JSON
// Set the default version of PowerShell to use
"powershell.powerShellExePath": "C:/Program Files/PowerShell/6/pwsh.exe",
// Keep focus in the editor when you execute script with F8
"powershell.integratedConsole.focusConsoleOnExecute": false,
// Enable Tab completion
"editor.tabCompletion": "on",
// Optional: Show whitespace characters in the editor
"editor.renderWhitespace": "all",
// Optional: Automatically trim trailing whitespace from files when saving
"files.trimTrailingWhitespace": true
```

To apply the changes, open the command pallette with <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>P</kbd>, type `Reload` and press <kbd>Enter</kbd>

Now you're ready to go!