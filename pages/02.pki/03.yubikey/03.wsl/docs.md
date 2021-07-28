---
title: 'WSL Setup for Yubikey'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - gpg
        - yubikey
        - wsl
---

# Configuring WSL2 for Yubikey Pass-Through

## Setup The Host

Install the pre-requisites:
* Install [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/)


Configure SSH Key and Git Integration With Windows 10 Native Way (Thanks to this [DevGenius](https://blog.devgenius.io/set-up-ssh-key-and-git-integration-in-windows-10-native-way-c9b94952dd2c) blog).

In an administrative Powershell prompt:

```powershell
Add-Content $env:APPDATA\gnupg\gpg-agent.conf "enable-putty-support"
Add-Content $env:APPDATA\gnupg\gpg-agent.conf "enable-ssh-support"

$OpenSSHClient = Get-WindowsCapability -Online | ? Name -like 'OpenSSH.Client*'
Add-WindowsCapability -Online -Name $OpenSSHClient.Name

$SSHAgentSvc = Get-Service -Name 'ssh-agent'
Set-Service -Name $SSHAgentSvc.Name -StartupType Automatic
Start-Service -Name $SSHAgentSvc.Name
```

In your normal user Powershell prompt:

```powershell
ssh-keygen # only if you don't already have an SSH key
ssh-add
```

## Inside a WSL2 Session
Thanks to [Jaroslav Živný's](https://dev.to/dzerycz/the-ultimate-guide-to-yubikey-on-wsl2-part-1-5aed) blog articles

```bash
sudo apt install socat
mkdir ~/.ssh
wget https://github.com/BlackReloaded/wsl2-ssh-pageant/releases/download/v1.2.0/wsl2-ssh-pageant.exe -O ~/.ssh/wsl2-ssh-pageant.exe
chmod +x ~/.ssh/wsl2-ssh-pageant.exe
```

Copy this into a script that will be run at session startup, `~/.gpg4wsl`

```bash
# SSH Socket
# Removing Linux SSH socket and replacing it by link to wsl2-ssh-pageant socket
export SSH_AUTH_SOCK=$HOME/.ssh/agent.sock 
ss -a | grep -q $SSH_AUTH_SOCK 
if [ $? -ne 0 ]; then
  rm -f $SSH_AUTH_SOCK
  setsid nohup socat UNIX-LISTEN:$SSH_AUTH_SOCK,fork EXEC:$HOME/.ssh/wsl2-ssh-pageant.exe &>/dev/null &
fi
# GPG Socket
# Removing Linux GPG Agent socket and replacing it by link to wsl2-ssh-pageant GPG socket
export GPG_AGENT_SOCK=$HOME/.gnupg/S.gpg-agent 
ss -a | grep -q $GPG_AGENT_SOCK 
if [ $? -ne 0 ]; then
  rm -rf $GPG_AGENT_SOCK
  setsid nohup socat UNIX-LISTEN:$GPG_AGENT_SOCK,fork EXEC:"$HOME/.ssh/wsl2-ssh-pageant.exe --gpg S.gpg-agent" &>/dev/null &
fi
```

Add this to the end of the `~/.bash_rc` or `~/.zshrc` script, or wherever you want to auto-run the script from:

```bash
source ~/.gpg4wsl
```

Back on the host PC, restart WSL:
```
wsl.exe --shutdown
```

The next time you start a WSL session, you should be able to get some info out of `gpg --card-status`
