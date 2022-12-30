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

## Initialize and Configure Umee node

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

## Setup Seeds

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
```console
# change <wallet_orch> to yours
umeed keys add <wallet_orch>
```
```console
# change <wallet_pfd> to yours
umeed keys add <wallet_pfd>
```

Recover Keplr acoount. Use your `mnemonic` phrase. 
Send token ``UMEE`` to it. 


## Setup Peggo

```console
git clone https://github.com/umee-network/peggo.git
```
```console
cd peggo
```
Take the latest version:
```console
git checkout v1.4.0
```
```console
make install
```
To check:
```console
peggo version
```
Output:
```
version: v1.4.0
commit: 30f87edb6f73e659ec12fba465e00aaa414e137b
sdk: v0.46.7-umee
go: go1.19.1 linux/amd64
```

## Var
```console
UMEE_CHAIN="umee-1"
```
```console
UMEE_HOME="$HOME/.umee"
```
```console

```console
MAIN_ADDR=$(umeed keys show <wallet> -a --home /root/.umee)
```
```console
echo $MAIN_ADDR
```
```console
echo 'export MAIN_ADDR='${MAIN_ADDR} >> $HOME/.bash_profile
```
```console
ORCH_ADDR=$(umeed keys show <wallet_orch> -a --home $UMEE_HOME)
```
```console
echo $ORCH_ADDR
```
```console
echo 'export ORCH_ADDR='${ORCH_ADDR} >> $HOME/.bash_profile
```
```console
UMEE_VALOPER=$(umeed keys show $MAIN_WALLET --bech val -a --home $UMEE_HOME)
```
```console
echo $UMEE_VALOPER
```
```console
echo 'export UMEE_VALOPER='${UMEE_VALOPER} >> $HOME/.bash_profile
```
```console
BRIDGE_ADDR="0xb564ac229e9d6040a9f1298b7211b9e79ee05a2c"
```
```console
# Private key of PEGGO_ETH_ADDR (0x7d318xxxxxxxxxxxxxxxxxxxxxxxxxxxx)
PEGGO_ETH_PK="78bcffc6a73dxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```
```console
ETH_RPC="https://eth-mainnet.g.alchemy.com/v2/0HLixxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```
```console
ALCHEMY_ENDPOINT="wss://eth-mainnet.g.alchemy.com/v2/0HLixxxxxxxxxxxxxxxxxxxxxxxxx"
```
```console
ORCHESTRATOR_WALLET_NAME="<wallet_orch>"
```
```console
KEYRING_PASSWORD="<your wallet_orch password>"
```

## Service file Peggo

```console
nano /etc/systemd/system/peggo.service
```
Output (copy and paste):
```console
[Unit]
Description=Peggo Service
After=network.target

[Service]
User=root
Type=simple
ExecStart=/root/gopath/bin/peggo orchestrator 0xb564ac229e9d6040a9f1298b7211b9e79ee05a2c \
  --eth-rpc="https://eth-mainnet.g.alchemy.com/v2/0HLixxxxxxxxxxxxxxxxxx" \
  --relay-batches=true \
  --valset-relay-mode=minimum \
  --cosmos-chain-id=umee-1 \
  --cosmos-keyring-dir="/root/.umee" \
  --cosmos-from=<wallet_orch> \
  --cosmos-from-passphrase=<password> \
  --oracle-providers="osmosis,huobi,okx,coinbase" \
  --eth-alchemy-ws="wss://eth-mainnet.g.alchemy.com/v2/0HLixxxxxxxxxxxxxxxxxxx" \
  --cosmos-keyring="os" \
  --log-level debug \
  --log-format text
Environment="PEGGO_ETH_PK=b8a5exxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

Let's try to start ``peggo``. First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```

Then enable the ``peggo`` service:
```console
sudo -S systemctl enable peggo
```

We can now start ``peggo``:
```console
sudo systemctl start peggo
```

We can check that the service is running:
```console
sudo systemctl status peggo
```

Check logs:
```console
journalctl -f -n 200 -u peggo
```

## Register orchestrator keys
```console
umeed tx gravity set-orchestrator-address <umee_valoper> <wallet_orch_addr> <PEGGO_ETH_ADDR> --from <wallet> --chain-id umee-1  --fees 3000uumee --home /root/.umee
```

## Install price-feeder
```console
cd $HOME/umee/price-feeder
```
```console
git checkout price-feeder/v2.0.2
```
```console
make install
```
```console
mkdir -p $HOME/price-feeder_config
```
## Configure price-feeder
```console
nano $HOME/price-feeder_config/price-feeder.toml
```
Output:
```
gas_adjustment = 1

[server]
listen_addr = "0.0.0.0:7171"
read_timeout = "20s"
verbose_cors = true
write_timeout = "20s"

[[deviation_thresholds]]
base = "USDT"
threshold = "1.5"

[[deviation_thresholds]]
base = "UMEE"
threshold = "1.5"

[[deviation_thresholds]]
base = "ATOM"
threshold = "1.5"

[[currency_pairs]]
base = "UMEE"
providers = [
  "okx",
  "osmosis",
  "mexc"
]
quote = "USDT"

[[currency_pairs]]
base = "UMEE"
providers = [
  "okx",
  "osmosis",
  "mexc"
]
quote = "USD"

[[currency_pairs]]
base = "USDT"
providers = [
  "kraken",
  "okx",
  "coinbase"
]
quote = "USD"

[[currency_pairs]]
base = "ATOM"
providers = [
  "okx",
  "osmosis",
  "mexc"
]
quote = "USDT"

[[currency_pairs]]
base = "ATOM"
providers = [
  "okx",
  "osmosis",
  "mexc"
]
quote = "USD"

[account]
address = <wallet_pfd_addr>
chain_id = umee-1
validator = <umee_valoper>

[keyring]
backend = "os"
dir = "/root/.umee"
pass = "${KEYRING_PASSWORD}"

[rpc]
grpc_endpoint = "localhost:${GRPC_PORT}"
rpc_timeout = "100ms"
tmrpc_endpoint = "http://localhost:${RPC_PORT}"

[telemetry]
enable-hostname = true
enable-hostname-label = true
enable-service-label = true
enabled = false
global_labels = [["chain-id", "umee-1"]]
service-name = "pfd"
prometheus-retention-time = 100

[[provider_endpoints]]
name = "binance"
rest = "https://api1.binance.com"
websocket = "stream.binance.com:9443"
```
Create service:
```console
nano /etc/systemd/system/pfd.service
```
Output:
```
[Unit]
Description=Price feeder service
After=network.target
[Service]
User=root
Environment=PRICE_FEEDER_PASS=${KEYRING_PASSWORD}
Type=simple
ExecStart=/root/gopath/bin/price-feeder /root/price-feeder_config/price-feeder.toml --log-level debug
RestartSec=10
Restart=on-failure
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
```

Let's try to start ``pfd``. First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```

Then enable the ``pfd`` service:
```console
sudo -S systemctl enable pfd
```

We can now start ``pfd``:
```console
sudo systemctl start pfd
```

We can check that the service is running:
```console
sudo systemctl status pfd
```

Check logs:
```console
journalctl -f -n 200 -u pfd
```

## Delegate price-feeder address
```console
umeed tx oracle delegate-feed-consent <wallet_addr> <wallet_pfd_addr> --fees 3000uumee --chain-id umee-1
```

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
