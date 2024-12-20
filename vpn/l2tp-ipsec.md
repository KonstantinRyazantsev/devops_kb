How to setup L2TP/IPSEC VPN server on Ubuntu. Tested on Ubuntu 24.04

create `docker-compose.yaml` file:

```yaml
version: "3.5"
services:
  ipsec-vpn-server:
    image: hwdsl2/ipsec-vpn-server
    container_name: ipsec-vpn-server
    env_file: ./vpn.env
    restart: always
    privileged: true
    ports:
      - "500:500/udp"
      - "4500:4500/udp"
    volumes:
      - ./ikev2-vpn-data:/etc/ipsec.d
      - /lib/modules:/lib/modules:ro
```

create `.env` file: like

```
VPN_IPSEC_PSK='secret'
VPN_USER='vpnuser'
VPN_PASSWORD='default user password'
VPN_ADDL_USERS=vpnuser1 vpnuser2
VPN_ADDL_PASSWORDS=vpnuser1_password vpnuser2_password
```

execute:

```bash
sudo apt update && sudo apt -y install htop sysstat iftop iotop jq
sudo curl -fsSL https://get.docker.com/ | sh
VER_DOCK=`curl -s  https://api.github.com/repos/docker/compose/releases|jq -r ".[].tag_name,.[].prerelease"|grep -v 'rc'|sed -n '1p'`
sudo curl -L https://github.com/docker/compose/releases/download/$VER_DOCK/docker-compose-`uname -s`-`uname -m` -o /usr/bin/docker-compose
sudo chmod +x /usr/bin/docker-compose
sudo usermod -aG docker $USER

docker-compose down && docker-compose up -d
```

