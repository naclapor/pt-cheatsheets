## NMAP Basics
---
### TCP

```shell
sudo nmap -sC -sV -O <target>

sudo nmap -sC -sV -O -p- <target>
```

### UDP

```shell
sudo nmap -sU <target>
```

## NMAP Single Host Scans
---
### Initial Scan

```bash
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial $IP
```

### Full Scan

```bash
sudo nmap -sC -sV -O -p- -oA /usr/share/nmap/full $IP
```

### Fast TCP port scan

```bash
nmap -F -T4 -oN nmap/fastTCPScan $IP
```

### Aggressive scan

```bash
nmap -A -T4 -px,y,z -v -oN nmap/aggressiveScan $IP
```

### TCP Version detection
```bash
nmap -sV --reason -O -p- $IP
```

### UDP Version detection

```bash
nmap -sU -sV -n $IP
```

### Heartbleed Check

```bash
nmap --script ssl-heartbleed $IP
```

### Version/OS detection

```bash
nmap -v --dns-server \<DNS\> -sV --reason -O --open -Pn $IP
```

### Unknown Services

```bash
amap -d $IP \<PORT\>
```

### Full vulnerability scanning

```bash
nmap -sS -sV --script=/path/to/your/vulnscan.nse -oN nmap/vulnScan $IP
```

### Full _nmapAutomator_ scanning

```bash
sudo nmapAutomator.sh --host $IP --type All
```

## NMAP Subnet Scans
---
### List scan

```bash
nmap -sL -oN nmap/listScan 10.x.x.x/xx
```

### Ping scan (run it with privileges)

```bash
nmap -sn -oN nmap/pingScan 10.x.x.x/xx
```

### Hosts discovery

```bash
netdiscover -r 10.x.x.x/24
```