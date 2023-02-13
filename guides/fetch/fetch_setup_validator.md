# Fetch.ai
``Setup validator node manually``

## install dependencies, if needed
```console
sudo apt update
```
```console
sudo apt install -y curl git jq lz4 build-essential htop
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
## Install Fetch.ai

Clone the fetchd github repository:
```console
git clone https://github.com/fetchai/fetchd
```

Go to ``cd fetchd`` directory:
```console
cd fetchd
```

Take new release:
```console
git checkout v0.10.5
```

Compile and install:
```console
make install
```
Check that the installation was successful:
```console
fetchd version
```

This should print the version number of fetchd.

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
mkdir -p ~/.fetchd/cosmovisor/genesis/bin
```
```console
mkdir -p ~/.fetchd/cosmovisor/upgrades
```

Then copy the ``fetchd`` binanry to the ``genesis/bin`` folder:
```console
cp ~/go/bin/fetchd ~/.fetchd/cosmovisor/genesis/bin
```

## Initialize and Configure fetch node

Now let's initialize the node:
```console
NODE_MONIKER=<YOUR_NODE_MONIKER>
```

```console
fetchd init NODE_MONIKER --chain-id fetchhub-4
```

## Download the latest Genesis File

Download the latest genesis file:
```console
wget -O genesis.json https://snapshots.polkachu.com/genesis/fetch/genesis.json --inet4-only
```

```console
mv genesis.json ~/.fetchd/config
```

## Setup Seeds

Script for you to update ``seeds`` setting with these peers in ``config.toml``
```console
sed -i 's/seeds = ""/seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:15256"/' ~/.fetchd/config/config.toml
```
Minimum-gas-prices
```console
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0afet"|g' $HOME/.fetchd/config/app.toml
```

Check. Don't be lazy:
```console
cat config.toml
```

## Setup pruning

Settings:
```console
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.fetchd/config/app.toml
```
```console
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.fetchd/config/app.toml
```
```console
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.fetchd/config/app.toml
```
```console
sed -i 's/snapshot-interval *=.*/snapshot-interval = 0/g' $HOME/.fetchd/config/app.toml
```

Pruning:
```
pruning-keep-recent = "100"
pruning-keep-every = "0"
pruning-interval = "10"
```

## Setup Snapshot polkachu.com
fresh snapshot you can always find [here](https://polkachu.com/tendermint_snapshots/fetch)

Download the snapshot:
```console
sudo apt install snapd -y
```
```console
sudo snap install lz4
```
```console
wget -O fetch_9797958.tar.lz4 https://snapshots.polkachu.com/snapshots/fetch/fetch_9797958.tar.lz4 --inet4-only
```

Decompress the snapshot to your database location:
```console
lz4 -c -d fetch_9797958.tar.lz4  | tar -x -C $HOME/.fetchd
```

Remove downloaded snapshot to free up space:
```console
rm -v fetch_9797958.tar.lz4
```

## Setup Service file

Use editor ``nano``:
```console
nano /etc/systemd/system/fetch.service
```

Output:
```
[Unit]
Description="fetch node"
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=fetchd"
Environment="DAEMON_HOME=/root/.fetchd"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```

Let's try to start ``fetchd``. First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```

Then enable the ``fetchd`` service:
```console
sudo -S systemctl enable fetchd
```

We can now start ``fetchd``:
```console
sudo systemctl start fetchd
```

We can check that the service is running:
```console
sudo systemctl status fetchd
```

Check logs:
```console
journalctl -f -n 200 -u fetchd
```
## Addr book

```console 
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/fetch/addrbook.json --inet4-only
```
```console
mv addrbook.json ~/.fetchd/config
```

## Wallet preparation:

Let's generate a new key:
```console
# change <wallet> to yours
fetchd keys add <wallet>
```

Recover Keplr acoount. Use your `mnemonic` phrase. 
Send token ``FET`` to it. 

## Setup Validator

Make transaction. to change the ``moniker``, ``from`` and ``chain-id`` and other values as required:
```console
# change <moniker>, <keybase>, <website>, <details>, <wallet> to yours
fetchd tx staking create-validator \
  --commission-max-change-rate 0.01 \
  --commission-max-rate 0.20 \
  --commission-rate 0.05 \
  --amount 1000000000000000000afet \
  --pubkey $(fetchd tendermint show-validator) \
  --chain-id fetchhub-4 \
  --min-self-delegation "1" \
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
  fetchd tx slashing unjail --from <wallet> --chain-id fetchhub-4 --gas-adjustment 1.5 --gas auto
  ```
  
Governance vote:
  ```console
  fetchd tx gov vote 10 yes --from <wallet> --chain-id fetchhub-4 --gas-adjustment 1.5 --gas auto -y
  ```

Check block height:
```console
curl -s localhost:26657/status | jq .result.sync_info.latest_block_height
```

Withdraw Commission And Rewards From Your Validator
```console
fetchd tx distribution withdraw-rewards $(fetchd keys show <wallet> --bech val -a) --commission --from <wallet> --chain-id fetchhub-4 --gas-adjustment 1.5 --gas auto -y
```

Delegate to yourself (Self bonded)
```console
fetchd tx staking delegate $(fetchd keys show <wallet> --bech val -a) 1000000000000000000afet --from <wallet> --chain-id fetchhub-4 --gas-adjustment 1.5 --gas auto -y
```
