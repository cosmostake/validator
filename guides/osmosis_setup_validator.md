# Osmosis
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
## Install Osmosis

Clone the Osmosis github repository:
```console
git clone https://github.com/osmosis-labs/osmosis
```

Go to ``osmosis`` directory:
```console
cd osmosis
```

Take new release:
```console
git checkout v11.0.0
```

Compile and install:
```console
make install
```
Check that the installation was successful:
```console
osmosisd version
```

This should print the version number of osmosisd.

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
mkdir -p $HOME/.osmosisd/cosmovisor/genesis/bin
```
```console
mkdir $HOME/.osmosisd/cosmovisor/upgrades
```

Then copy the ``osmosisd`` binanry to the ``genesis/bin`` folder:
```console
cp $GOPATH/bin/osmosisd $HOME/.osmosisd/cosmovisor/genesis/bin
```
Now we need to add some environment variables that Cosmovisor is dependent on:
```console
export DAEMON_HOME=$HOME/.osmosisd
export DAEMON_RESTART_AFTER_UPGRADE=true
export DAEMON_ALLOW_DOWNLOAD_BINARIES=false
export DAEMON_NAME=osmosisd
export UNSAFE_SKIP_BACKUP=true
```

Check everything done:
```console
cosmovisor version
```

Output 
```
11.0.0
```

This should print the version number of osmosisd.

## Initialize and Configure Sifnode

Now let's initialize the node:
```console
NODE_MONIKER=<YOUR_NODE_MONIKER>
```
```console
osmosisd config chain-id osmosis-1
```
```console
osmosisd init $NODE_MONIKER --chain-id osmosis-1
```

## Download the latest Genesis File

Download the latest genesis file:
```console
curl https://github.com/osmosis-labs/networks/raw/main/osmosis-1/genesis.json > $HOME/.osmosisd/config/genesis.json
```

Check the genesis file:
```console
sha256sum $HOME/.osmosisd/config/genesis.json
```

## Setup Seeds and Persistance Peers

First do:
```console
seeds="21d7539792ee2e0d650b199bf742c56ae0cf499e@162.55.132.230:2000,295b417f995073d09ff4c6c141bd138a7f7b5922@65.21.141.212:2000,ec4d3571bf709ab78df61716e47b5ac03d077a1a@65.108.43.26:2000,4cb8e1e089bdf44741b32638591944dc15b7cce3@65.108.73.18:2000,f515a8599b40f0e84dfad935ba414674ab11a668@osmosis.blockpane.com:26656,6bcdbcfd5d2c6ba58460f10dbcfde58278212833@osmosis.artifact-staking.io:26656"
```
```console
peers="83c06bc290b6dffe05aa9cec720bedfc118afcbc@osmosis.nodejumper.io:35656"
```
Script for you to update ``persistent_peers`` and ``seeds`` setting with these peers in ``config.toml``
```console
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.osmosisd/config/config.toml
```

Check. Don't be lazy:
```console
cat config.toml
```

## Setup pruning

Settings:
```console
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.osmosisd/config/app.toml
```
```console
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.osmosisd/config/app.toml
```
```console
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.osmosisd/config/app.toml
```
```console
sed -i 's/snapshot-interval *=.*/snapshot-interval = 0/g' $HOME/.osmosisd/config/app.toml
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
wget -O osmosis_5901913.tar.lz4 https://snapshots1.polkachu.com/snapshots/osmosis/osmosis_5901913.tar.lz4
```

Decompress the snapshot to your database location:
```console
lz4 -c -d osmosis_5901913.tar.lz4  | tar -x -C $HOME/.osmosisd
```

Remove downloaded snapshot to free up space:
```console
rm -v osmosis_5901913.tar.lz4
```

## Setup Service file

Use editor ``nano``:
```console
nano /etc/systemd/system/osmosisd.service
```

Output:
```
[Unit]
Description=Osmosis Node
After=network-online.target

[Service]
User=$USER
ExecStart=/root/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=osmosisd"
Environment="DAEMON_HOME=/root/.osmosisd"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```

Let's try to start ``osmosisd``. First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```

Then enable the ``osmosisd`` service:
```console
sudo -S systemctl enable osmosisd
```

We can now start ``osmosisd``:
```console
sudo systemctl start osmosisd
```

We can check that the service is running:
```console
sudo systemctl status osmosisd
```

Check logs:
```console
journalctl -f -n 200 -u osmosisd
```

## Wallet preparation:

Let's generate a new key:
```console
# change <wallet> to yours
osmosisd keys add <wallet> --keyring-backend file
```

Recover Keplr acoount. Use your `mnemonic` phrase. 
Send token ``OSMO`` to it. 

