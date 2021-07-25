---
title: 'Git Setup for Commit Signing'
taxonomy:
    category:
        - PKI
    tag:
        - pki
        - gpg
        - yubikey
        - git
---

# Using Yubikeys to sign git commits

## Powershell

Install the pre-requisites:
* Install [GPG4Win](https://gpg4win.org/download.html)
* Install the [YubiKey SmartCard Minidriver](https://www.yubico.com/support/download/smart-card-drivers-tools/)


Configure SSH Key and Git Integration With Windows 10 Native Way (Thanks to this [DevGenius](https://blog.devgenius.io/set-up-ssh-key-and-git-integration-in-windows-10-native-way-c9b94952dd2c) blog):
```powershell
D:\work> $OpenSSHClient = Get-WindowsCapability -Online | ? Name -like 'OpenSSH.Client*'
D:\work> Add-WindowsCapability -Online -Name $OpenSSHClient.Name
D:\work> 
D:\work> $SSHAgentSvc = Get-Service -Name ‘ssh-agent’
D:\work> Set-Service -Name $SSHAgentSvc.Name -StartupType Automatic
D:\work> Start-Service -Name $SSHAgentSvc.Name
D:\work> 
D:\work> ssh-keygen
D:\work> ssh-add
```

Find the ID of the GPG key to use:
```
D:\work> gpg --list-keys
-----------------------------------------------
pub   rsa4096 2021-07-24 [C]
      EF7C2A657A623928FBFD1E762D3F69C229E9DA52
uid           [ultimate] Your Name <someone@somewhere.nice>

pub   rsa4096 2021-07-24 [C]
      EA14073E363E13426E02460A12FC86C2B6B4F011
uid           [ultimate] Your Name <someone@somewhere.nice>
sub   rsa4096 2021-07-24 [S] [expires: 2022-07-24]
sub   rsa4096 2021-07-24 [E] [expires: 2022-07-24]
sub   rsa4096 2021-07-24 [A] [expires: 2022-07-24]
D:\work> 
```
Make a note of the long hex id above the three sub-keys (`EA14...`)

Configure git: 
```
D:\work> git config --global gpg.program "C:\Program Files (x86)\GnuPG\bin\gpg.exe"
D:\work> git config --global user.email someone@somewhere.nice
D:\work> git config --global user.name "Your Name"
D:\work> git config --global user.signingkey EA14073E363E13426E02460A12FC86C2B6B4F011
D:\work> git config --global commit.gpgsign true
```

Confirm that git is using the correct executables. This shouldn't error but should return some GPG output:
```
D:\work> &(git config gpg.program) --list-keys
```

## WSL2
Thanks to [Jaroslav Živný's](https://dev.to/dzerycz/the-ultimate-guide-to-yubikey-on-wsl2-part-1-5aed) blog articles

On the host PC from a Command prompt:
```cmd
D:\work> choco install gnupg putty.install
D:\work> mkdir %APPDATA%\gnupg
D:\work> echo enable-putty-support >> %APPDATA%\gnupg\gpg-agent.conf
D:\work> echo enable-ssh-support >> %APPDATA%\gnupg\gpg-agent.conf
D:\work> gpg --card-status
```

Then from within a WSL2 (Ubuntu) session:
```bash
sudo apt install socat
mkdir ~/.ssh
wget https://github.com/BlackReloaded/wsl2-ssh-pageant/releases/download/v1.2.0/wsl2-ssh-pageant.exe -O ~/.ssh/wsl2-ssh-pageant.exe
chmod +x ~/.ssh/wsl2-ssh-pageant.exe
```

Add this to the end of the `~/.bash_rc` or `~/.zshrc` script:
```bash
# GPG Socket
# Removing Linux GPG Agent socket and replacing it by link to wsl2-ssh-pageant GPG socket
export GPG_AGENT_SOCK=$HOME/.gnupg/S.gpg-agent 
ss -a | grep -q $GPG_AGENT_SOCK 
if [ $? -ne 0 ]; then
  rm -rf $GPG_AGENT_SOCK
  setsid nohup socat UNIX-LISTEN:$GPG_AGENT_SOCK,fork EXEC:"$HOME/.ssh/wsl2-ssh-pageant.exe --gpg S.gpg-agent" &>/dev/null &
fi
```


Restart WSL from the host:
```
wsl.exe --shutdown
```

The next time you start a WSL session, you should be able to get some info out of `gpg --card-status`

Find the ID of the GPG key to use:
```
D:\work> gpg --list-keys
-----------------------------------------------
pub   rsa4096 2021-07-24 [C]
      EF7C2A657A623928FBFD1E762D3F69C229E9DA52
uid           [ultimate] Your Name <someone@somewhere.nice>

pub   rsa4096 2021-07-24 [C]
      EA14073E363E13426E02460A12FC86C2B6B4F011
uid           [ultimate] Your Name <someone@somewhere.nice>
sub   rsa4096 2021-07-24 [S] [expires: 2022-07-24]
sub   rsa4096 2021-07-24 [E] [expires: 2022-07-24]
sub   rsa4096 2021-07-24 [A] [expires: 2022-07-24]
D:\work> 
```
Make a note of the long hex id above the three sub-keys (`EA14...`)

Configure git: 
```bash
git config --global gpg.program gpg
git config --global user.email someone@somewhere.nice
git config --global user.name "Your Name"
git config --global user.signingkey EA14073E363E13426E02460A12FC86C2B6B4F011
git config --global commit.gpgsign true
```
