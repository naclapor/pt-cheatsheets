## SMB (139-445)
---

### Check for share

```shell
smbmap -H <target>
```

```shell
smbclient -N -L //<target>

smbclient -N -L \\\\<target>\\
```

### Connect to share

```shell
smbclient //<target>

smbclient \\\\<target>\\

smbclient //<target>/<share> -U domain.com/username%password

smbclient //<target>/<share> -U username --pw-nt-hash <hash> -W domain.com
```

## SNMP (161-162)
---

```shell
snmpwalk -c public -v1 -t 5 <target>

# Users
snmpwalk -c public -v1 <target> 1.3.6.1.4.1.77.1.2.25

# Running Processes
snmpwalk -c public -v1 <target> 1.3.6.1.2.1.25.4.2.1.2

# Installed Software
snmpwalk -c public -v1 <target> 1.3.6.1.2.1.25.6.3.1.2

# Open TCP ports
snmpwalk -c public -v1 <target> 1.3.6.1.2.1.6.13.1.3
```
