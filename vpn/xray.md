How to setup XRay VLESS with Reality TLS VPN server on Ubuntu. Tested on Ubuntu 24.04

XRay supports other protocols: VMESS, Trojan and others, but they are not covered by this guide

details: 
* https://github.com/XTLS/Xray-core

Install docker-compose if needed:

```bash
sudo apt update && sudo apt -y install htop sysstat iftop iotop jq
sudo curl -fsSL https://get.docker.com/ | sh
VER_DOCK=`curl -s  https://api.github.com/repos/docker/compose/releases|jq -r ".[].tag_name,.[].prerelease"|grep -v 'rc'|sed -n '1p'`
sudo curl -L https://github.com/docker/compose/releases/download/$VER_DOCK/docker-compose-`uname -s`-`uname -m` -o /usr/bin/docker-compose
sudo chmod +x /usr/bin/docker-compose
sudo usermod -aG docker $USER
```

Create `docker-compose.yaml` file:

```yaml
version: '3'
services:
  xray:
    image: teddysun/xray
    container_name: xray
    volumes:
      - ./config.json:/etc/xray/config.json
    ports:
      - "443:443"
    restart: always
```

Create `config.json` file: like

```json
{
  "log": {
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "PUT_RANDOM_CLIENT_UUID_HERE",
            "flow": "xtls-rprx-vision",
            "alterId": 64,
            "security": "auto",
            "level": 0,
            "remarks": "PUT_HUMAN_READABLE_CLIENT_DESCRIPTION_HERE"            
          }          
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "show": false,
          "dest": "www.cloudflare.com:443", // Any other server name can be used here to mimic traffic as it goes there
          "serverNames": [
            "www.cloudflare.com" // Should correspond to "dest"
          ],
          "privateKey": "PUT_YOUR_REALITY_PRIVATE_KEY",
          "shortIds": [
            "PUT_SHORT_ID_PER_CLIENT_HERE"
            ],
          "password": "PUT_YOUR_RANDOM_PASSWORD_HERE"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom"
    }
  ]
}
```

Generate Reality keys:

```bash
docker run --rm teddysun/xray xray x25519 > vless-reality.keys
cat vless-reality.keys
```

Generate a user ID:

```bash
uuidgen
```

Generate a user short ID:
```bash
 curl -s "https://www.random.org/cgi-bin/randbyte?nbytes=8&format=h" | tr -d ' ' && echo
```

Generate Reality password:
```bash
 < /dev/urandom tr -dc 'A-Za-z0-9' | head -c20; echo
```

Run XRay:

```
docker-compose up -d
```

Manage users by editing `config.json` `inbounds.settings.clients` section and add a new `inbounds.streamSettings.shortIds` for each client.

VPN Clients:

* Android: https://play.google.com/store/apps/details?id=com.v2ray.ang
* Windows: https://github.com/2dust/v2rayN

  Client configuration hints:

  * `Address` - Your XRay server IP
  * `Port` - `443`
  * `ID` - `Id` of the user specified in the `config.json`
  * `Flow` - `xtls-rprx-vision`
  * `Encryption` - `none`
  * `Network` - `tcp`
  * `Header/Camouflage type` - `none`
  * `HTTP node/Camouflage domain` - keep empty
  * `Path` - keep empty
  * `TLS` - `reality`
  * `SNI` - `www.cloudflare.com`
  * `Fingerprint` - select whatever you prefer
  * `Public key` - get the public key from your `vless-reality.keys` file
  * `ShortID` - Select one of the `ShortIds` specified in the `config.json`
  * `SpiderX` - keep empty

Configure Keenetic router to use XRay

* https://www.youtube.com/watch?v=rKbENP80ECo
* https://rockblack.su/vpn/dopolnitelno/vless-keenetic
* https://habr.com/ru/articles/760052/
