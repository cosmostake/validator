# Persistence
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
## Install Persistence

Clone the Persistence github repository:
```console
git clone https://github.com/persistenceOne/persistenceCore
```

Go to ``persistence`` directory:
```console
cd persistenceCore
```

Take new release:
```console
git checkout v5.0.0
```

Compile and install:
```console
make install
```
Check that the installation was successful:
```console
persistenceCore version
```

This should print the version number of persistenceCore.

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
mkdir -p $HOME/.persistenceCore/cosmovisor/genesis/bin
```
```console
mkdir $HOME/.persistenceCore/cosmovisor/upgrades
```

Then copy the ``persistenceCore`` binanry to the ``genesis/bin`` folder:
```console
cp $GOPATH/bin/persistenceCore $HOME/.persistenceCore/cosmovisor/genesis/bin
```
Now we need to add some environment variables that Cosmovisor is dependent on:
```console
export DAEMON_HOME=$HOME/.persistenceCore
export DAEMON_RESTART_AFTER_UPGRADE=true
export DAEMON_ALLOW_DOWNLOAD_BINARIES=false
export DAEMON_NAME=persistenceCore
export UNSAFE_SKIP_BACKUP=true
```

Check everything done:
```console
cosmovisor version
```

This should print the version number of persistenceCore.

## Initialize and Configure Persistence node

Now let's initialize the node:
```console
NODE_MONIKER=<YOUR_NODE_MONIKER>
```
```console
persistenceCore config chain-id core-1
```
```console
persistenceCore init $NODE_MONIKER --chain-id core-1
```

## Download the latest Genesis File

Download the latest genesis file:
```console
curl https://raw.githubusercontent.com/persistenceOne/networks/master/core-1/final_genesis.json > $HOME/.persistenceCore/config/genesis.json
```

Check the genesis file:
```console
sha256sum $HOME/.persistenceCore/config/genesis.json
```

## Setup Seeds and Persistance Peers

First do:
```console
seeds="646d0ad08c408f93276f90cd29d4e410e2d60f63@xprt.paranorm.pro:25656,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:15456"
```
```console
peers="646d0ad08c408f93276f90cd29d4e410e2d60f63@xprt.paranorm.pro:25656"
```
Script for you to update ``persistent_peers`` and ``seeds`` setting with these peers in ``config.toml``
```console
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.persistenceCore/config/config.toml
```
Minimum-gas-prices
```console
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.005uxprt"|g' $HOME/.persistenceCore/config/app.toml
```

Check. Don't be lazy:
```console
cat config.toml
```

## Setup pruning

Settings:
```console
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.persistenceCore/config/app.toml
```
```console
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.persistenceCore/config/app.toml
```
```console
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.persistenceCore/config/app.toml
```
```console
sed -i 's/snapshot-interval *=.*/snapshot-interval = 0/g' $HOME/.persistenceCore/config/app.toml
```

Pruning:
```
pruning-keep-recent = "100"
pruning-keep-every = "0"
pruning-interval = "10"
```

## Setup Snapshot polkachu.com
fresh snapshot you can always find [here](https://polkachu.com/tendermint_snapshots/persistence)

Download the snapshot:
```console
sudo apt install snapd -y
```
```console
sudo snap install lz4
```
```console
wget -O persistence_9254954.tar.lz4 https://snapshots.polkachu.com/snapshots/persistence/persistence_9254954.tar.lz4 --inet4-only
```

Decompress the snapshot to your database location:
```console
lz4 -c -d persistence_9254954.tar.lz4  | tar -x -C $HOME/.persistenceCore
```

Remove downloaded snapshot to free up space:
```console
rm -v persistence_9254954.tar.lz4
```

## Setup Service file

Use editor ``nano``:
```console
nano /etc/systemd/system/persistenceCore.service
```

Output:
```
[Unit]
Description=Persistence Node
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=persistenceCore"
Environment="DAEMON_HOME=/root/.persistenceCore"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```

Let's try to start ``persistenceCore``. First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```

Then enable the ``persistenceCore`` service:
```console
sudo -S systemctl enable persistenceCore
```

We can now start ``persistenceCore``:
```console
sudo systemctl start persistenceCore
```

We can check that the service is running:
```console
sudo systemctl status persistenceCore
```

Check logs:
```console
journalctl -f -n 200 -u persistenceCore
```
## Addr book

```console 
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/persistence/addrbook.json --inet4-only
```
```console
mv addrbook.json ~/.persistenceCore/config
```

## Wallet preparation:

Let's generate a new key:
```console
# change <wallet> to yours
persistenceCore keys add <wallet>
```

Recover Keplr acoount. Use your `mnemonic` phrase. 
Send token ``XPRT`` to it. 

## Setup Validator

Make transaction. to change the ``moniker``, ``from`` and ``chain-id`` and other values as required:
```console
# change <moniker>, <keybase>, <website>, <details>, <wallet> to yours
persistenceCore tx staking create-validator \
  --commission-max-change-rate 0.01 \
  --commission-max-rate 0.20 \
  --commission-rate 0.05 \
  --amount 1000000uxprt \
  --pubkey $(persistenceCore tendermint show-validator) \
  --chain-id core-1 \
  --min-self-delegation "1" \
  --gas-prices 0.005uxprt \
  --gas-adjustment 1.5 \
  --gas auto \
  --moniker <moniker> \
  --identity <keybase> \
  --website <website> \
  --details <details> \
  --from <wallet>
  ```
Unjail:
  ```console
  persistenceCore tx slashing unjail --from <wallet> --chain-id core-1 --gas-prices 0.005uxprt --gas-adjustment 1.5 --gas auto
  ```
  
Governance vote:
  ```console
  persistenceCore tx gov vote 10 yes --from <wallet> --chain-id core-1 --gas-prices 0.005uxprt --gas-adjustment 1.5 --gas auto -y
  ```

Check block height:
```console
curl -s localhost:20657/status | jq .result.sync_info.latest_block_height
```
