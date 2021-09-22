---
title: 'HomeLab Jump Server'
taxonomy:
    category:
        - homelab
        - jump
    tag:
        - homelab
        - ssh
        - yubikey
---


[mermaid]
flowchart LR
    Firefox-- SOCKS5 -->PuTTY
    PuTTY-- SSH -->id1(Firewall)
    Firewall-- SSH -->Jump Server
[/mermaid]

## In the HomeLab

### Networks
Forward some external port through to the server that will be the SSH jump server.
On a Mikrotik this is probably just a NAT rule and a port rule:

```routeros
/ip firewall nat add action=dst-nat chain=dstnat comment="SSH Jump" dst-port=22 in-interface=ether1 log=yes log-prefix=ssh protocol=tcp to-addresses=172.29.14.196 to-ports=22

/ip firewall filter add action=accept chain=forward comment="SSH Jump" dst-port=22 in-interface=ether1 log=yes log-prefix=ssh protocol=tcp
```
Adjust for the correct IP, port and interfaces as appropriate.

### Jump Server
Edit the `/etc/ssh/sshd_config` file and only allow cert-based authentication. You might want to do this *after* you've managed to get the public key into the authorized_keys file, if you don't have other methods to get onto the server.
```
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
UsePAM no
```
Restart sshd.


## On the client

### Install:
  * [PuTTY CAC](https://github.com/NoMoreFood/putty-cac/releases)
  * [OpenSC](https://github.com/OpenSC/OpenSC/wiki)
  * [FoxyProxy](https://addons.mozilla.org/en-GB/firefox/addon/foxyproxy-standard/)

### Set up the certificates:
1. Plug in the Yubikey
1. In the Yubikey Manager generate a new certificate in the Applications>PIV tool.  
   Certificate type: **Self-signed**  
   Algorithm: **ECCP384**  
   Subjet: **anything** really, like your name  
   Expires: **way in the future**
1. **Either** use openssh to convert the public key:
    1. In the GUI export the public key (e.g. `yubikey.pem`)
    1. Convert the exported key to an x509 public key:  
       ```
       openssl x509 -pubkey -in yubikey.pem -noout > yubikey.pub
       ```
    1. Concert the public key to a PKCS8 standard that PuTTY understands:  
       ```
       ssh-keygen -i -f yubikey.pub -mPKCS8 > yubikey.openssh.pub
       ```
1. **Or** use the OpenSC tools to extract the key:
   1. Open PowerShell and  
      ```
      PS> CD C:\Program Files\OpenSC Project\OpenSC\tools
      ```
   1. Determine the "slot" that the Yubikey has been nominated (in this case, `2`):  
      ```
      PS> pkcs11-tool.exe --list-slots
      Slot 2 (0x9): Yubico YubiKey OTP+FIDO+CCID 0
      ```
   1. Determine the corret key to use (in this case ID is `01`):  
      ```
      PS> .\pkcs15-tool.exe --reader 2 -k
      Private EC Key [PIV AUTH key]
      Object Flags   : [0x01], private
      Usage          : [0x04], sign
      Access Flags   : [0x1D], sensitive, alwaysSensitive, neverExtract, local
      Algo_refs      : 0
      FieldLength    : 384
      Key ref        : 154 (0x9A)
      Native         : yes
      Auth ID        : 01
      ID             : 01
      MD:guid        :  0x'43......023220000000000000000'
      ```
   1. Finally, extract the pubkey:
      ```
      PS> .\pkcs15-tool.exe --reader 2 --read-ssh-key 01
      ecdsa-sha2-nistp384 AAAAE2VjZHNhLXNoYdItbmlzdHAzODQAAAAIbmlzdHAzODQAAABhBIsN6+cMvpGvqDHbfcG1hjN5xL75yf+++76D7AlE9GYMs3VrIQXL9serER9qCrjZNxhldK/J6sFB/QWivmCcgqqKaHoIhew0dtKM037QWM/BdSvZ0ZupPNZCLcsu7IC7og== PIV AUTH pubkey
      ```
1. Append that public key to the `~/.ssh/authorized_keys` file on the jump server

### Setup Pageant
1. Launch pageant
1. On the taskbar, right-click the Pageant icon and click Add PKCS Cert
1. Browse to `C:\Program Files\OpenSC Project\OpenSC\pkcs11` and select `opensc-pkcs11.dll`
1. In the window "PuTTY: Select Certificate for Authentication" click "more choices"
1. The recently-generated key should be in the list, select it

### Setup PuTTY
1. Launch PuTTY
1. Create a new stored session (e.g. `Homelab`)
   * Connection -> Data: Auto-login username: `your remote username`
   * Connection -> SSH -> Tunnels:  
     Source Port: `45543` (or any convenient free local port)  
     Destination: `blank`  
     Type: `Dynamic`
   * Window -> Colours:
     Change `ANSI Blue` to something not ridiculously dark like the default. `166 166 255` works quite nicely

### Setup Firefox

Once connected to the SSH session it will be possible to utilise the connection as a SOCKS5 proxy.
In Firefox this is made easy using the [FoxyProxy](https://addons.mozilla.org/en-GB/firefox/addon/foxyproxy-standard/) extension.

1. Install the extension
1. Open the extension config from the toolbar button
1. Add a new Proxy:  
      Title: `HomeLabSSH`
      Proxy Type: `SOCKS5`
      IP: `127.0.0.1`
      Port: `45543` (or whatever was put into the PuTTY config earlier)
      Username: `blank`
      Password: `blank`
1. Add some appropriate patterns to route traffic automatically:  
      Name: `Home Domain`, Pattern: `*.home.cylindric.net`  
      Name: `Home Domain (all ports)`, Pattern: `*.home.cylindric.net:*`  
      Name: `Home IP`, Pattern: `172.29.14.*`  
      Name: `Home IP (all ports)`, Pattern: `172.29.14.*:*`
1. On the browser button, select "Use enabled proxies by pattern"
