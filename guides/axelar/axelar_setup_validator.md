# DRAFT! Axelar
``Setup validator node manually``

## install dependencies, if needed
```console
sudo apt update
```
```console
sudo apt install -y curl git jq lz4 build-essential
```
## Install Go

If you need to update or install 'go' we'd recommend to use the handy go version manager called g. To install 'go' with it use:
```console
curl -sSL https://git.io/g-install | sh -s
```
```console
source ~/.bashrc
```

You can test that Go is installed by executing:
```console
go version
```
## Install Axelar

Clone the Axelar github repository:
```console
git clone https://github.com/axelarnetwork/axelarate-community.git
```

Go to ``axelarate-community`` directory:
```console
cd axelarate-community
```

Compile and install:
```console
./scripts/setup-node.sh -n mainnet
```

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
mkdir -p -v $HOME/.axelar/cosmovisor/genesis/bin
```
```console
mkdir -v $HOME/.axelar/cosmovisor/upgrades
```

Then copy the ``axelard`` binanry to the ``genesis/bin`` folder:
```console
cp -v $HOME/.axelar/bin/axelard-v0.26.0 $HOME/.axelar/cosmovisor/genesis/bin
```
Now we need to add some environment variables that Cosmovisor is dependent on:
```console
export DAEMON_HOME=$HOME/.axelar
export DAEMON_RESTART_AFTER_UPGRADE=true
export DAEMON_ALLOW_DOWNLOAD_BINARIES=false
export DAEMON_NAME=axelard-v0.26.0
export UNSAFE_SKIP_BACKUP=true
```

Check everything done:
```console
cosmovisor version
```

Output 
```
0.26.0
```

This should print the version number of axelard.

## Initialize and Configure Axelar node

Now let's initialize the node:
```console
NODE_MONIKER=<YOUR_NODE_MONIKER>
```
```console
axelard config chain-id axelar-dojo-1
```
```console
axelard init $NODE_MONIKER --chain-id axelar-dojo-1
```

## Download the latest Genesis File

Download the latest genesis file:
```console
curl https://share.blockpane.com/juno/phoenix/genesis.json > $HOME/.juno/config/genesis.json
```

Check the genesis file:
```console
sha256sum $HOME/.juno/config/genesis.json
```

## Setup Seeds and Persistance Peers

First do:
```console
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:15156"
```
```console
peers="608f7b9f7c6f9b1df26c9b737183a60dc09fc4cf@95.217.116.178:26656,fa17c9818e8bd3e616ad3b7c1b7b697b16d079ea@65.108.103.184:24656,24bc6c4d546c71b09f3d69f911690049d1cf56b1@66.70.177.20:26656,fc08e2e9b1ffd5941a07896a8b79cf85db5e8da9@18.119.69.44:26656,7914a1eec502d75f98ab5552ff678d71c01e0b34@34.162.61.253:26656,b6a69792c3ef83b79ee53e47d5bb183ac987ee75@193.35.56.166:26656,1bd159908c6385a5c9767711c389cbe5d2027b1d@89.149.218.182:26656,d1d9761737f3008bfa6e48f356ea4f40073ef4da@84.244.95.227:26656,17eeb3bc6963a834bd5e39c6101085990baeecf5@54.157.26.234:26656,353f7d0962594bcbfb63c81594e35e39c4c89a1a@18.223.127.165:26656,67d5010dbb99376a5eba920fc596660fb9c71180@146.59.0.123:2060,658a8a6a82227b345584decd902c763f81a64d6a@23.88.3.237:26656,7136c52148b37f2cbc182be9c93c93ef89e54c81@46.4.91.49:31656"
```
Script for you to update ``persistent_peers`` and ``seeds`` setting with these peers in ``config.toml``
```console
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.axelar/config/config.toml
```
Set `minimum-gas-prices`:
```console
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.007uaxl\/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9"|g' $HOME/.axelar/config/app.toml
```

Check. Don't be lazy:
```console
cd .axelar/config && cat config.toml
```

## Setup pruning

Settings:
```console
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.axelar/config/app.toml
```
```console
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.axelar/config/app.toml
```
```console
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.axelar/config/app.toml
```

Pruning:
```
pruning-keep-recent = "100"
pruning-keep-every = "0"
pruning-interval = "10"
```

## Setup Snapshot Polkachu.com
Current version of snapshot always avaible [here](https://polkachu.com/tendermint_snapshots/axelar)
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
wget -O axelar_4065307.tar.lz4 https://snapshots.polkachu.com/snapshots/axelar/axelar_4065307.tar.lz4 --inet4-only
```

Decompress the snapshot to your database location:
```console
lz4 -c -d axelar_4065307.tar.lz4  | tar -x -C $HOME/.axelar
```

Remove downloaded snapshot to free up space:
```console
rm -v axelar_4065307.tar.lz4
```

## Setup Service file

Use editor ``nano``:
```console
nano /etc/systemd/system/junod.service
```

Output:
```
[Unit]
Description=Axelar Node
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=axelard-v0.26.0"
Environment="DAEMON_HOME=/root/.axelar"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```

Let's try to start ``axelard``. First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```

Then enable the ``axelard`` service:
```console
sudo -S systemctl enable axelard
```

We can now start ``axelard``:
```console
sudo systemctl start axelard
```

We can check that the service is running:
```console
sudo systemctl status axelard
```

Check logs:
```console
journalctl -f -n 200 -u axelard
```

## Wallet preparation:

Let's generate a new key:
```console
/root/.axelar/bin/axelard keys add validator --home /root/.axelar
```
```console
/root/.axelar/bin/axelard keys add broadcaster --home /root/.axelar
```

Recover Keplr acoount. Use your `mnemonic` phrase. 
Send token ``AXL`` to it. 

## Setup Validator

```console
cd axelarate-community
```
```console
./scripts/setup-validator.sh -n mainnet
```
```console
/root/.axelar/bin/tofnd -m create -d /root/.axelar/tofnd
```

Make transaction. to change the ``moniker``, ``from`` and ``chain-id`` and other values as required:
```console
# change <wallet> to yours
junod tx staking create-validator \
  --commission-max-change-rate 0.01 \
  --commission-max-rate 0.20 \
  --commission-rate 0.05 \
  --amount 1000000ujuno \
  --pubkey $(junod tendermint show-validator) \
  --chain-id juno-1 \
  --min-self-delegation "1" \
  --gas-prices 0.1ujuno \
  --gas-adjustment 1.5 \
  --gas auto \
  --moniker "COIN SIDE" \
  --identity <your_keybase> \
  --website="https://github.com/COIN-SIDE/validator" \
  --details="Our crypto community aspires to a decentralized future" \
  --from <wallet>
  ```
Unjail
  ```console
  junod tx slashing unjail --from <wallet> --chain-id juno-1 --gas-prices 0.1ujuno --gas-adjustment 1.5 --gas auto -y
  ```
  
  Governance vote
  ```console
  junod tx gov vote 30 yes --from <wallet> --chain-id juno-1 --gas-prices 0.1ujuno --gas-adjustment 1.5 --gas auto -y
  ```
