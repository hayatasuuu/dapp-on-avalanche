# dapp-on-avalanche

## Ref

- https://github.com/ovieokeh/avalanche-dapp-tutorial
- https://blog.logrocket.com/build-dapp-avalanche-complete-guide/

## Installation

```bash
$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.6 LTS
Release:        18.04
Codename:       bionic
$ git version
git version 2.17.1
```

### STEP1 : Install Golang

Latest version : [https://go.dev/dl/](https://go.dev/dl/)

```bash
# $ wget https://go.dev/dl/go1.17.6.linux-amd64.tar.gz
# $ sudo tar -C /usr/local -xzf go1.17.6.linux-amd64.tar.gz 
$ wget https://go.dev/dl/go1.17.6.linux-arm64.tar.gz
$ sudo tar -C /usr/local -xzf go1.17.6.linux-arm64.tar.gz 
$ export PATH=$PATH:/usr/local/go/bin
$ go version
go version go1.17.6 linux/arm64
$ rm go1.17.6.linux-arm64.tar.gz 
```

### STEP2 : Install Node.js

```bash
$ sudo apt update
$ curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
$ sudo apt install -y nodejs
$ node --version
v16.13.2
```

### STEP3 : Install Avalanchego

```
$ sudo su -
wget -O - https://downloads.avax.network/avalanchego.gpg.key | apt-key add -
echo "deb https://downloads.avax.network/apt bionic main" > /etc/apt/sources.list.d/avalanche.list
exit
$ sudo apt update
$ sudo apt install avalanchego
```

### Step4 : Connecting a Local Testnet

```bash
$ avalanchego --network-id=local --staking-enabled=false --snow-sample-size=1 --snow-quorum-size=1
```

