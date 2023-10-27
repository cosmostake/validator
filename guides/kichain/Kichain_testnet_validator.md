# Kichain TESTNET
``Setup validator testnet node manually``

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
git checkout 5.0.2-beta
```

Compile and install:
```console
make build-testnet
```
Or
```console
wget https://github.com/KiFoundation/ki-tools/releases/download/5.0.2-beta/kid-testnet-5.0.2-beta-linux-amd64
```
Check that the installation was successful:
```console
kid version
```

This should print the version number of kid.

## Initialize and Configure Ki node

Now let's initialize the node:
```console
NODE_MONIKER=<YOUR_NODE_MONIKER>
```
```console
kid config chain-id kichain-t-4
```
```console
kid init $NODE_MONIKER --chain-id kichain-t-4
```

## Download the latest Genesis File

Download the latest genesis file:
```console
wget -O genesis.json https://snapshots.polkachu.com/testnet-genesis/kichain/genesis.json --inet4-only
```
```console
mv -v genesis.json ~/.kid/config
```

## Setup Seeds and Persistance Peers

First do:
```console
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:13556"
```
Script for you to update ``seeds`` setting with these peers in ``config.toml``
```console
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|' $HOME/.kid/config/config.toml
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
wget -O kichain_12145507.tar.lz4 https://snapshots.polkachu.com/testnet-snapshots/kichain/kichain_12145507.tar.lz4 --inet4-only
```

Decompress the snapshot to your database location:
```console
lz4 -c -d kichain_12145507.tar.lz4  | tar -x -C $HOME/.kid
```

Remove downloaded snapshot to free up space:
```console
rm -v kichain_12145507.tar.lz4
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
ExecStart=/root/go/bin/kid start
Restart=always
RestartSec=3
LimitNOFILE=10000

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
  --amount 9900000000utki \
  --pubkey $(kid tendermint show-validator) \
  --chain-id kichain-t-4 \
  --min-self-delegation "1" \
  --gas-prices 0.25utki \
  --gas-adjustment 1.5 \
  --gas auto \
  --moniker "COSMØSTAKE" \
  --identity "" \
  --website "" \
  --details "" \
  --from ""
  ```
