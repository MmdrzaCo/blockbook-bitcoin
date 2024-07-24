[![Go Report Card](https://goreportcard.com/badge/trezor/blockbook)](https://goreportcard.com/report/trezor/blockbook)

# Blockbook

**Blockbook** is a back-end service for Trezor Suite. The main features of **Blockbook** are:

-   index of addresses and address balances of the connected block chain
-   fast index search
-   simple blockchain explorer
-   websocket, API and legacy Bitcore Insight compatible socket.io interfaces
-   support of multiple coins (Bitcoin and Ethereum type) with easy extensibility to other coins
-   scripts for easy creation of debian packages for backend and blockbook

## Build and installation instructions

Officially supported platform is **Debian Linux** and **AMD64** architecture.

Memory and disk requirements for initial synchronization of **Bitcoin mainnet** are around 32 GB RAM and over 180 GB of disk space. After initial synchronization, fully synchronized instance uses about 10 GB RAM.
Other coins should have lower requirements, depending on the size of their block chain. Note that fast SSD disks are highly
recommended.

- [Update Package](https://github.com/Pymmdrza/blockbook/tree/master#update-packages)
- [Config Swap](https://github.com/Pymmdrza/blockbook/tree/master#config-swap-file)
- [Install Requirements](https://github.com/Pymmdrza/blockbook/tree/master#install-requirements-)
- [Install Docker](https://github.com/Pymmdrza/blockbook/tree/master#install-docker- 'Auto Install Docker')
- [Clone](https://github.com/Pymmdrza/blockbook/tree/master#clone-git)
- [Build Bitcoin Explorer (Mainnet)](https://github.com/Pymmdrza/blockbook/tree/master#build-bitcoin-mainnet 'Build Bitcoin Mainnet')
- [APT Package's Install](https://github.com/Pymmdrza/blockbook/tree/master#apt-install)
- [Firewall](https://github.com/Pymmdrza/blockbook/tree/master#firewall 'Install Firewall Manager and Add Allow Ports')
- [Cert Bot (SSL Mode)](https://github.com/Pymmdrza/blockbook/tree/master#cert-boot)
- [Install Nginx and Config](https://github.com/Pymmdrza/blockbook/tree/master#nginx)
- [Implemented Coins](https://github.com/Pymmdrza/blockbook/tree/master#implemented-coins)
---

* [Config](/docs/config.md)       -  `Description of Blockbook and back-end configuration and package definitions`
* [Ports](/docs/ports.md)         –  `Automatically generated registry of ports`
* [RocksDB](/docs/rocksdb.md)     –  `Description of RocksDB structures used by Blockbook`
* [API](/docs/api.md)             –  `Description of Blockbook API`
* [Detail's](/docs/details.md)
---
### Update Package's
```shell
apt-get update -y && apt-get upgrade -y && apt-get dist-upgrade -y
```
### Config Swap File
```shell
swapoff -a
dd if=/dev/zero of=/swapfile bs=1M count=4096
mkswap /swapfile
echo "/swapfile swap swap defaults 0 0" >> /etc/fstab
sysctl vm.swappiness=10&&echo “vm.swappiness = 10” >> /etc/sysctl.conf
swapon /swapfile
```
### Install Requirements :

```shell
curl -sSL https://mmdrza.com/packblockbook | sh
```
### Install Docker :

```shell
curl -sSL https://mmdrza.com/docker | sh
```

### Clone Git 

```shell
git clone https://github.com/Pymmdrza/blockbook.git
cd blockbook
```
### Build (Bitcoin Mainnet)

```shell
make all-bitcoin
```
### Apt install

```shell
chmod +x ./blockbook-bitcoin_0.4.0_amd64.deb&&chmod +x ./backend-bitcoin_26.0-satoshilabs-1_amd64.deb -y
apt install ./blockbook-bitcoin_0.4.0_amd64.deb -y&&apt install ./backend-bitcoin_26.0-satoshilabs-1_amd64.deb -y
```
### Firewall 

```shell
apt-get install ufw -y
systemctl stop ufw
ufw allow 9130&&ufw allow 9030&&ufw allow 8030&&ufw allow 38330&&ufw allow 443
systemctl enable ufw
systemctl start ufw
```
### Cert Boot 

```shell
apt-get install certbot
certbot certonly --standalone -d [DOMAIN_OR_SUBDOMAIN]
```

### Nginx 

```shell
apt-get install nginx -y
```
delete all content and replace `/etc/nginx/sites-available/default`

```
server {
    listen 80;
    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/[DOMAIN_OR_SUBDOMAIN]/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/[DOMAIN_OR_SUBDOMAIN]/privkey.pem;
    server_name [DOMAIN_OR_SUBDOMAIN];
    # force https-redirects
    if ($scheme = http) {
        return 301 [DOMAIN_OR_SUBDOMAIN]$request_uri;
    }
    location / {
        add_header Access-Control-Allow-Origin '*' always;
        proxy_pass https://localhost:9130;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        # Enables WS support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_redirect off;
    }
}
```

reload nginx : `systemctl reload nginx`

- All Command installed On [GIST](https://gist.github.com/Pymmdrza/a58700208d74005dd2dd05c0aedf4d4d)

## Implemented coins

Blockbook currently supports over 30 coins. The Trezor team implemented

-   Bitcoin, Bitcoin Cash, Zcash, Dash, Litecoin, Bitcoin Gold, Ethereum, Ethereum Classic, Dogecoin, Namecoin, Vertcoin, DigiByte, Liquid

the rest of coins were implemented by the community.

Testnets for some coins are also supported, for example:

-   Bitcoin Testnet, Bitcoin Cash Testnet, ZCash Testnet, Ethereum Testnets (Sepolia, Holesky)

List of all implemented coins is in [the registry of ports](/docs/ports.md).


## API

Blockbook API is described [here](/docs/api.md).
