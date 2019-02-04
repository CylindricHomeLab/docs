---
title: 'PowerShell Tips'
taxonomy:
  category:
      - Windows
---

Various tips about PowerShell...

===

# Simple TCP Listener

This is useful for testing inbound network connectivity.

```ps
function Listen-Port ($port=80) {
<#
.DESCRIPTION
Temporarily listen on a given port for connections dumps connections to the screen - useful for troubleshooting
firewall rules.

.PARAMETER Port
The TCP port that the listener should attach to

.EXAMPLE
PS C:\> listen-port 443
Listening on port 443, press CTRL+C to cancel

DateTime                                      AddressFamily Address                                                Port
--------                                      ------------- -------                                                ----
3/1/2016 4:36:43 AM                            InterNetwork 192.168.20.179                                        62286
Listener Closed Safely

.INFO
Created by Shane Wright. Neossian@gmail.com

#>
    $endpoint = New-Object System.Net.IPEndPoint ([system.net.ipaddress]::any, $port)    
    (new-object System.Net.Sockets.TcpListener (new-object System.Net.IPEndPoint ([system.net.ipaddress]::any, $port))).Start()

    $listener.Server.ReceiveTimeout = 3000
    $listener.Start()    
    try {
    Write-Host "Listening on port $port, press CTRL+C to cancel"
    While ($true) {
        if (!$listener.Pending())
        {
            Start-Sleep -Seconds 1;
            Continue;
        }
        $client = $listener.AcceptTcpClient()
        $client.client.RemoteEndPoint | Add-Member -NotePropertyName DateTime -NotePropertyValue (get-date) -PassThru
        $client.Close()
        }
    }
    catch {
        Write-Error $_          
    }
    finally{
            $listener.Stop()
            Write-host "Listener Closed Safely"
    }

}
```
