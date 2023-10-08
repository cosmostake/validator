# Kichain
``Setup validator node manually``

## install dependencies, if needed
```console
sudo apt update
```
```console
sudo apt install -y curl git jq lz4 build-essential
```
## Install Go

To install 'go' with it use:
```console
sudo rm -rvf /usr/local/go/
```
```console
wget https://golang.org/dl/go1.19.10.linux-amd64.tar.gz
```
```console
sudo tar -C /usr/local -xzf go1.19.10.linux-amd64.tar.gz
```
```console
rm go1.19.10.linux-amd64.tar.gz
```

## Configure Go
```console
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
```

You can test that Go is installed by executing:
```console
go version
```
## Install Kichain

Clone the Osmosis github repository:
```console
git clone https://github.com/KiFoundation/ki-tools kichain
```

Go to ``kichain`` directory:
```console
cd kichain
```

Take new release:
```console
git checkout 5.0.2
```

Compile and install:
```console
make install
```

Check that the installation was successful:
```console
kid version
```

This should print the version number of kid.

## Setup Cosmovisor

Go back:
```console
cd
```

Clone the cosmos-sdk github repository:
```console
git clone https://github.com/cosmos/cosmos-sdk.git
```

Go to ``cosmos-sdk`` directory:
```console
cd cosmos-sdk
```

Take new release:
```console
git checkout v0.45.8
```

Compile and install:
```console
make cosmovisor
```

Then copy the ``cosmovisor`` binanry to the ``genesis/bin`` folder:
```console
cp cosmovisor/cosmovisor $GOPATH/bin/cosmovisor
```
```console
cd $HOME
```

## Configure Cosmovisor

First, create the required directories:
```console
mkdir -p $HOME/.kid/cosmovisor/genesis/bin
```
```console
mkdir $HOME/.kid/cosmovisor/upgrades
```

Then copy the ``kid`` binanry to the ``genesis/bin`` folder:
```console
cp $GOPATH/bin/kid $HOME/.kid/cosmovisor/genesis/bin
```
Now we need to add some environment variables that Cosmovisor is dependent on:
```console
export DAEMON_HOME=$HOME/.kid
export DAEMON_RESTART_AFTER_UPGRADE=true
export DAEMON_ALLOW_DOWNLOAD_BINARIES=false
export DAEMON_NAME=kid
export UNSAFE_SKIP_BACKUP=true
```

Check everything done:
```console
cosmovisor version
```

Output 
```
Mainnet-3.0.0
```

This should print the version number of kid.

## Initialize and Configure Ki node

Now let's initialize the node:
```console
NODE_MONIKER=<YOUR_NODE_MONIKER>
```
```console
kid config chain-id kichain-2
```
```console
kid init $NODE_MONIKER --chain-id kichain-2
```

## Download the latest Genesis File

Download the latest genesis file:
```console
curl https://raw.githubusercontent.com/KiFoundation/ki-networks/v0.1/Mainnet/kichain-2/genesis.json > $HOME/.kid/config/genesis.json
```

Check the genesis file:
```console
sha256sum $HOME/.kid/config/genesis.json
```

## Setup Seeds and Persistance Peers

First do:
```console
seeds="24cbccfa8813accd0ebdb09e7cdb54cff2e8fcd9@51.89.166.197:26656"
```
```console
peers="766ed622c79fa9cfd668db9741a1f72a5751e0cd@kichain.nodejumper.io:28656"
```
Script for you to update ``persistent_peers`` and ``seeds`` setting with these peers in ``config.toml``
```console
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.kid/config/config.toml
```

Also minimum gas prices:
```console
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001uxki"|g' $HOME/.kid/config/app.toml
```

Check. Don't be lazy:
```console
cat config.toml
```

## Setup pruning

Settings:
```console
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.kid/config/app.toml
```
```console
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.kid/config/app.toml
```
```console
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.kid/config/app.toml
```

Pruning:
```
pruning-keep-recent = "100"
pruning-keep-every = "0"
pruning-interval = "10"
```

## Setup Snapshot Polkachu.com

Install lz4:
```console
sudo apt install snapd -y
```
```console
sudo snap install lz4
```

Download the snapshot:
```console
cd
```
```console
wget -O kichain_17383816.tar.lz4 https://snapshots.polkachu.com/snapshots/kichain/kichain_17383816.tar.lz4 --inet4-only
```

Decompress the snapshot to your database location:
```console
lz4 -c -d kichain_17383816.tar.lz4  | tar -x -C $HOME/.kid
```

Remove downloaded snapshot to free up space:
```console
rm -v kichain_17383816.tar.lz4
```

## Setup Service file

Use editor ``nano``:
```console
nano /etc/systemd/system/kid.service
```

Output:
```
[Unit]
Description=Kichain Node
After=network-online.target

[Service]
User=$USER
ExecStart=/root/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=kid"
Environment="DAEMON_HOME=/root/.kid"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```

Let's try to start ``kid``. First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```

Then enable the ``kid`` service:
```console
sudo -S systemctl enable kid
```

We can now start ``kid``:
```console
sudo systemctl start kid
```

We can check that the service is running:
```console
sudo systemctl status kid
```

Check logs:
```console
journalctl -f -n 200 -u kid
```

## Wallet preparation:

Let's generate a new key:
```console
# change <wallet> to yours
kid keys add <wallet> --keyring-backend file
```

Recover Keplr acoount. Use your `mnemonic` phrase. 
Send token ``XKI`` to it. 

## Setup Validator

Make transaction. to change the ``moniker``, ``from`` and ``chain-id`` and other values as required:
```console
# change <wallet> to yours
kid tx staking create-validator \
  --commission-max-change-rate 0.01 \
  --commission-max-rate 0.20 \
  --commission-rate 0.05 \
  --amount 10000000uxki \
  --pubkey $(kid tendermint show-validator) \
  --chain-id kichain-2 \
  --min-self-delegation "1" \
  --gas-prices 0.1uxki \
  --gas-adjustment 1.5 \
  --gas auto \
  --moniker "COIN SIDE" \
  --identity <your_keybase> \
  --website "https://github.com/COIN-SIDE/validator" \
  --details "Our crypto community aspires to a decentralized future" \
  --from <wallet> \
  --node "tcp://127.0.0.1:20657"
  ```
Unjail
  ```console
  kid tx slashing unjail --from <wallet> --chain-id kichain-2 --gas-prices 0.1uxki --gas-adjustment 1.5 --gas auto --node "tcp://127.0.0.1:20657"
  ```
  
  Governance vote
  ```console
  kid tx gov vote 98 yes --from <wallet> --chain-id kichain-2 --gas-prices 0.1uxki --gas-adjustment 1.5 --gas auto --node "tcp://127.0.0.1:20657" -y
  ```
