# KYVE
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
## Install KYVE

Clone the kyve github repository:
```console
git clone https://github.com/KYVENetwork/chain.git
```

Go to ``chain`` directory:
```console
cd chain
```

Take new release:
```console
git checkout v1.0.0
```

Compile and install:
```console
make install
```
Check that the installation was successful:
```console
kyved version
```

This should print the version number of kyved.

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
mkdir -v -p ~/.kyve/cosmovisor/genesis/bin
```
```console
mkdir -v -p ~/.kyve/cosmovisor/upgrades
```

Then copy the ``kyved`` binanry to the ``genesis/bin`` folder:
```console
cp -v ~/go/bin/kyved ~/.kyve/cosmovisor/genesis/bin
```

## Initialize and Configure kyve node

Now let's initialize the node:
```console
NODE_MONIKER=<YOUR_NODE_MONIKER>
```

```console
kyved init NODE_MONIKER --chain-id kyve-1
```

## Download the latest Genesis File

Download the latest genesis file:
```console
curl -Ls https://raw.githubusercontent.com/konsortech/Node/main/Mainnet/KYVE/genesis.json > $HOME/.kyve/config/genesis.json
```

## Setup Seeds and Peers

Script for you to update ``seeds`` setting with these peers in ``config.toml``
```console
SEEDS="4a72671447a1ede2a136dbac70ac4523c3433c94@seeds.cros-nest.com:29656"
```
```console
PEERS="b950b6b08f7a6d5c3e068fcd263802b336ffe047@18.198.182.214:26656,25da6253fc8740893277630461eb34c2e4daf545@3.76.244.30:26656,146d27829fd240e0e4672700514e9835cb6fdd98@34.212.201.1:26656,fae8cd5f04406e64484a7a8b6719eacbb861c094@44.241.103.199:26656,443f41172aafaa6c711333c621e019fde3f0ba99@5.75.144.137:26656,cfb5d3dc65e8e1d17285964655d2b47a44d35721@144.76.97.251:42656,c782ab00baf1c86261db0570307a9ecd9c5b197a@5.9.63.216:28656,38ef1bd70583c8831c3c0e07fad7e1bab56515cc@68.183.143.17:26656,307f4024107ef114dba355fe97dab44b8b45cefc@38.242.253.58:29656,86d313c22789ffa50c76b85b460f1e1412782a27@195.3.221.59:12656,a0ba3bd9616b51c26ab6ecc49a30a13d0438ab7f@65.109.94.250:28656,cec6c3c59d1bde0862d27500bf3c0ecc39b4727d@3.144.87.60:31309,0ab23bfd2924c09a0cb2166a78e65d6d0fbd172a@57.128.162.152:26656"
```
```console
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.kyve/config/config.toml
```
Minimum-gas-prices
```console
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.025ukyve\"|" $HOME/.kyve/config/app.toml
```

Check. Don't be lazy:
```console
cat config.toml
```
Disable indexing
```console
indexer="null"
```
```console
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.kyve/config/config.toml
```

## Setup pruning

Settings:
```console
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.kyve/config/app.toml
```
```console
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.kyve/config/app.toml
```
```console
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.kyve/config/app.toml
```
```console
sed -i 's/snapshot-interval *=.*/snapshot-interval = 0/g' $HOME/.kyve/config/app.toml
```

Pruning:
```
pruning-keep-recent = "100"
pruning-keep-every = "0"
pruning-interval = "10"
```

## Addr book

```console 
curl -Ls https://raw.githubusercontent.com/konsortech/Node/main/Mainnet/KYVE/addrbook.json > $HOME/.kyve/config/addrbook.json
```

## Setup Service file

Use editor ``nano``:
```console
nano /etc/systemd/system/kyved.service
```

Output:
```
[Unit]
Description="kyve node"
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/cosmovisor start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
Environment="DAEMON_NAME=kyved"
Environment="DAEMON_HOME=/root/.kyve"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```

Let's try to start ``kyved``. First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```

Then enable the ``kyved`` service:
```console
sudo -S systemctl enable kyved
```

We can now start ``kyved``:
```console
sudo systemctl start kyved
```

We can check that the service is running:
```console
sudo systemctl status kyved
```

Check logs:
```console
journalctl -f -n 200 -u kyved
```

## Wallet preparation:

Let's generate a new key:
```console
# change <wallet> to yours
kyved keys add <wallet>
```

Recover Keplr acoount. Use your `mnemonic` phrase. 
Send token ``KYVE`` to it. 

## Setup Validator

Make transaction. to change the ``moniker``, ``from`` and ``chain-id`` and other values as required:
```console
# change <moniker>, <keybase>, <website>, <details>, <wallet> to yours
kyved tx staking create-validator \
  --commission-max-change-rate 0.01 \
  --commission-max-rate 0.20 \
  --commission-rate 0.05 \
  --amount 1000000ukyve \
  --pubkey $(kyved tendermint show-validator) \
  --chain-id kyve-1 \
  --min-self-delegation "1" \
  --gas-adjustment 1.5 \
  --gas 51000000 \
  --fees=1100000ukyve \
  --moniker <moniker> \
  --identity <keybase> \
  --website <website> \
  --details <details> \
  --from <wallet>
  ```
Unjail:
  ```console
  kyved tx slashing unjail --from <wallet> --chain-id kyve-1 --gas-adjustment 1.5 --gas 51000000 --fees=1100000ukyve
  ```
  
Governance vote:
  ```console
  kyved tx gov vote 3 yes --from <wallet> --chain-id kyve-1 --gas-adjustment 1.5 --gas 51000000 --fees=1100000ukyve
  ```

Check block height:
```console
curl -s localhost:26657/status | jq .result.sync_info.latest_block_height
```

Withdraw Commission And Rewards From Your Validator
```console
kyved tx distribution withdraw-rewards $(kyved keys show <wallet> --bech val -a) --commission --from <wallet> --chain-id kyve-1 --gas-adjustment 1.5 --gas 51000000 --fees=1100000ukyve -y
```

Delegate to yourself (Self bonded)
```console
kyved tx staking delegate $(kyved keys show <wallet> --bech val -a) 1000000ukyve --from <wallet> --chain-id kyve-1 --gas-adjustment 1.5 --gas 51000000 --fees=1100000ukyve -y
```
