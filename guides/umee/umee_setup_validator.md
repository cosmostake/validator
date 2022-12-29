# Umee
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
## Install Umee

Clone the Umee github repository:
```console
git clone https://github.com/umee-network/umee
```

Go to ``umee`` directory:
```console
cd umee
```

Take new release:
```console
git checkout v3.3.0
```

Compile and install:
```console
make install
```

Check that the installation was successful:
```console
umeed version
```

This should print the version number of umeed.

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
mkdir -p -v $HOME/.umee/cosmovisor/genesis/bin
```
```console
mkdir -v $HOME/.umee/cosmovisor/upgrades
```

Then copy the ``umeed`` binanry to the ``genesis/bin`` folder:
```console
cp -v $GOPATH/bin/umeed $HOME/.umee/cosmovisor/genesis/bin
```

## Initialize and Configure Persistence node

Now let's initialize the node:
```console
# put your moniker name <YOUR_NODE_MONIKER>
NODE_MONIKER=<YOUR_NODE_MONIKER>
```

```console
umeed init $NODE_MONIKER --chain-id umee-1
```

## Download the latest Genesis File

Download the latest genesis file:
```console
curl https://raw.githubusercontent.com/umee-network/umee/main/networks/umee-1/genesis.json > $HOME/.umee/config/genesis.json
```

## Setup Seeds and Persistance Peers

Script for you to update ``seeds`` setting with these peers in ``config.toml``
```console
sed -i 's/seeds = ""/seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:13656"/' ~/.umee/config/config.toml
```
Download addrbook.json:
```console
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/umee/addrbook.json --inet4-only
```
```console
mv -v addrbook.json ~/.umee/config
```

Check. Don't be lazy:
```console
cat config.toml
```

## Setup pruning

Settings:
```console
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.umee/config/app.toml
```
```console
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.umee/config/app.toml
```
```console
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.umee/config/app.toml
```
```console
sed -i 's/snapshot-interval *=.*/snapshot-interval = 0/g' $HOME/.umee/config/app.toml
```

Pruning:
```
pruning-keep-recent = "100"
pruning-keep-every = "0"
pruning-interval = "10"
```

## Setup Snapshot polkachu.com
fresh snapshot you can always find [here](https://polkachu.com/tendermint_snapshots/umee)

Download the snapshot:
```console
sudo apt install snapd -y
```
```console
sudo snap install lz4
```
```console
wget -O umee_4524860.tar.lz4 https://snapshots.polkachu.com/snapshots/umee/umee_4524860.tar.lz4 --inet4-only
```

Decompress the snapshot to your database location:
```console
lz4 -c -d umee_4524860.tar.lz4  | tar -x -C $HOME/.umee
```

Remove downloaded snapshot to free up space:
```console
rm -v umee_4524860.tar.lz4
```

## Setup Service file

Use editor ``nano``:
```console
nano /etc/systemd/system/umeed.service
```

Output:
```
[Unit]
Description=Umee Node
After=network-online.target

[Service]
User=root
ExecStart=/root/gopath/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=umeed"
Environment="DAEMON_HOME=/root/.umee"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```

Let's try to start ``umeed``. First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```

Then enable the ``umeed`` service:
```console
sudo -S systemctl enable umeed
```

We can now start ``umeed``:
```console
sudo systemctl start umeed
```

We can check that the service is running:
```console
sudo systemctl status umeed
```

Check logs:
```console
journalctl -f -n 200 -u umeed
```

## Wallet preparation:

Let's generate a new key:
```console
# change <wallet> to yours
umeed keys add <wallet>
```

Recover Keplr acoount. Use your `mnemonic` phrase. 
Send token ``UMEE`` to it. 

## Setup Validator

Make transaction. to change the ``moniker``, ``from`` and ``chain-id`` and other values as required:
```console
# change <moniker>, <keybase>, <website>, <details>, <wallet> to yours
umeed tx staking create-validator \
  --commission-max-change-rate 0.01 \
  --commission-max-rate 0.20 \
  --commission-rate 0.05 \
  --amount 1000000uumee \
  --pubkey $(umeed tendermint show-validator) \
  --chain-id umee-1 \
  --min-self-delegation "1" \
  --gas-prices 0.00125uumee \
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
  umeed tx slashing unjail --from <wallet> --chain-id umee-1 --gas-prices 0.00125uumee --gas-adjustment 1.5 --gas auto
  ```
  
Governance vote:
  ```console
  umeed tx gov vote 10 yes --from <wallet> --chain-id umee-1 --gas-prices 0.00125uumee --gas-adjustment 1.5 --gas auto -y
  ```

Check block height:
```console
curl -s localhost:21657/status | jq .result.sync_info.latest_block_height
```

Withdraw Commission And Rewards From Your Validator
```console
umeed tx distribution withdraw-rewards $(umeed keys show <wallet> --bech val -a) --commission --from <wallet> --chain-id umee-1 --gas-prices 0.00125uumee --gas-adjustment 1.5 --gas auto -y
```

Delegate to yourself (Self bonded)
```console
umeed tx staking delegate $(umeed keys show <wallet> --bech val -a) 1000000uumee --from <wallet> --chain-id umee-1 --gas-prices 0.00125uumee --gas-adjustment 1.5 --gas auto -y
```

not for you. skip it!!!
```console
--node "tcp://127.0.0.1:21657"
```
