# Umee testnet
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
git checkout v4.1.0-rc4
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

## Initialize and Configure Umee node

Now let's initialize the node:
```console
# put your moniker name <YOUR_NODE_MONIKER>
NODE_MONIKER=<YOUR_NODE_MONIKER>
```

```console
umeed init $NODE_MONIKER --chain-id canon-2
```

## Download the latest Genesis File

Download the latest genesis file:
```console
curl https://raw.githubusercontent.com/umee-network/umee/main/networks/canon-2/genesis.json > $HOME/.umee/config/genesis.json
```

## Setup Seeds

Script for you to update ``seeds`` setting with these peers in ``config.toml``
```console
sed -i 's/seeds = ""/seeds = "ab9e8d7227a3199c2832018eec42ade5bf47e71d@35.215.72.45:26656,e89407a37d2ebe0dfa2291c5240abe3a5410995f@35.212.203.22:26656"/' ~/.umee/config/config.toml
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

## Setup Statesync
fresh statesync you can always find [here](https://explorer.network.umee.cc/canon-2/statesync)

```console
nano /root/.umee/config/config.toml
```
Output:
```
[statesync]
enable = true
rpc_servers = "https://rpc.ruby.canon-2.network.umee.cc:443,https://rpc.emerald.canon-2.network.umee.cc:443,https://rpc.sapphire.canon-2.network.umee.cc:443"
trust_height = 2288000
trust_hash = "b66e554558ba0a8c22904f10b632d340e6d3737fc87e7cd982bac331ac1c0062"
trust_period = "168h"  # 2/3 of unbonding time
```
```console
nano /root/.umee/config/app.toml
```
Output:
```
minimum-gas-prices = "0.1uumee"

[state-sync]
snapshot-interval=1000
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
ExecStart=/root/go/bin/umeed start
Restart=always
RestartSec=3
LimitNOFILE=10000

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
Faucet Goerli https://faucet.triangleplatform.com/ethereum/goerli
Faucet Canon-2 https://faucet.umee.cc/

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
UMEE_VALOPER=$(umeed keys show <wallet> --bech val -a --home $UMEE_HOME)
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
ALCHEMY_ENDPOINT="wss://eth-goerli.g.alchemy.com/v2/0HLixxxxxxxxxxxxxxxxxxxxxxxxx"
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
ExecStart=/root/go/bin/peggo orchestrator 0x4d6D7b1dF43C9dE926BeC5F733980Ad7da6D9486 \
  --eth-rpc="https://eth-goerli.g.alchemy.com/xxxxxxxxxx" \
  --relay-batches=true \
  --valset-relay-mode=minimum \
  --cosmos-chain-id=canon-2 \
  --cosmos-keyring-dir="/root/.umee" \
  --cosmos-from=<wallet_orch> \
  --cosmos-from-passphrase="password" \
  --oracle-providers="osmosis,huobi,okx,coinbase,osmosisv2,binance,binanceus,crypto" \
  --eth-alchemy-ws="wss://eth-goerli.g.alchemy.com/v2/bxxxxxxxxxx" \
  --cosmos-keyring="os" \
  --log-level debug \
  --log-format text
Environment="PEGGO_ETH_PK=630184582c0xxxxxxxxxx"
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
umeed tx gravity set-orchestrator-address <umee_valoper> <wallet_orch_addr> <PEGGO_ETH_ADDR> --from <wallet> --chain-id canon-2  --fees 30000uumee --home /root/.umee
```

## Install price-feeder
```console
cd $HOME/umee/price-feeder
```
```console
git checkout price-feeder/v2.1.0
```
```console
make install
```
```console
mkdir -v -p $HOME/price-feeder_config
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

[[deviation_thresholds]]
base = "USDC"
threshold = "1.5"

[[deviation_thresholds]]
base = "CRO"
threshold = "1.5"

[[deviation_thresholds]]
base = "DAI"
threshold = "2"

[[deviation_thresholds]]
base = "ETH"
threshold = "2"

[[deviation_thresholds]]
base = "WBTC"
threshold = "1.5"

[[deviation_thresholds]]
base = "BNB"
threshold = "2"

[[deviation_thresholds]]
base = "JUNO"
threshold = "2"

[[deviation_thresholds]]
base = "OSMO"
threshold = "2"

[[deviation_thresholds]]
base = "stATOM"
threshold = "2"

[[deviation_thresholds]]
base = "stOSMO"
threshold = "2"

[[deviation_thresholds]]
base = "IST"
threshold = "2"

[[currency_pairs]]
base = "UMEE"
providers = [
  "okx",
  "gate",
]
quote = "USDT"

[[currency_pairs]]
base = "UMEE"
providers = [
  "osmosisv2",
]
quote = "ATOM"

[[currency_pairs]]
base = "USDT"
providers = [
  "kraken",
  "coinbase",
  "binanceus",
]
quote = "USD"

[[currency_pairs]]
base = "ATOM"
providers = [
  "okx",
  "bitget",
]
quote = "USDT"

[[currency_pairs]]
base = "ATOM"
providers = [
  "kraken",
  "binanceus",
]
quote = "USD"

[[currency_pairs]]
base = "USDC"
providers = [
  "okx",
  "bitget",
  "kraken",
]
quote = "USDT"

[[currency_pairs]]
base = "DAI"
providers = [
  "okx",
  "bitget",
  "huobi",
]
quote = "USDT"

[[currency_pairs]]
base = "DAI"
providers = [
  "kraken",
]
quote = "USD"

[[currency_pairs]]
base = "ETH"
providers = [
  "okx",
  "bitget",
]
quote = "USDT"

[[currency_pairs]]
base = "ETH"
providers = [
  "kraken",
]
quote = "USD"

[[currency_pairs]]
base = "WBTC"
providers = [
  "okx",
  "bitget",
  "crypto",
]
quote = "USDT"

[[currency_pairs]]
base = "CRO"
providers = [
  "crypto",
  "bitget",
  "okx",
]
quote = "USDT"

[[currency_pairs]]
base = "BNB"
providers = [
  "binanceus",
  "bitget",
  "okx",
]
quote = "USDT"

[[currency_pairs]]
base = "OSMO"
providers = [
  "osmosisv2",
]
quote = "ATOM"

[[currency_pairs]]
base = "OSMO"
providers = [
  "bitget",
  "gate",
]
quote = "USDT"

[[currency_pairs]]
base = "OSMO"
providers = [
  "crypto",
]
quote = "USD"

[[currency_pairs]]
base = "stATOM"
providers = [
  "osmosisv2",
]
quote = "ATOM"

[[currency_pairs]]
base = "stOSMO"
providers = [
  "osmosisv2",
]
quote = "OSMO"

[[currency_pairs]]
base = "IST"
providers = [
  "osmosisv2",
]
quote = "OSMO"

[account]
address = "umee1xqe3w5drsdr4s70t2vvvvvvv"
chain_id = "canon-2"
validator = "umeevaloper12tysz6mzrawevvvvvvv"

[keyring]
backend = "test"
dir = "/root/.umee"
pass = "1942Xmaksimus"

[rpc]
grpc_endpoint = "localhost:9090"
rpc_timeout = "100ms"
tmrpc_endpoint = "http://localhost:26657"

[telemetry]
enable-hostname = true
enable-hostname-label = true
enable-service-label = true
enabled = true
global-labels = [["chain_id", "canon-2"]]
service-name = "price-feeder"
prometheus-retention-time = 100

[[provider_endpoints]]
name = "binance"
rest = "https://api1.binance.com"
websocket = "stream.binance.com:9443"

[[provider_endpoints]]
name = "osmosisv2"
rest = "https://api.osmo-api.prod.network.umee.cc"
websocket = "api.osmo-api.prod.network.umee.cc"


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
ExecStart=/root/go/bin/price-feeder /root/price-feeder_config/price-feeder.toml --log-level debug
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
umeed tx oracle delegate-feed-consent <wallet_addr> <wallet_pfd_addr> --fees 30000uumee --chain-id canon-2
```

## Setup Validator

Make transaction. to change the ``moniker``, ``from`` and ``chain-id`` and other values as required:
```console
# change <moniker>, <keybase>, <website>, <details>, <wallet> to yours
umeed tx staking create-validator \
  --commission-max-change-rate 0.01 \
  --commission-max-rate 0.20 \
  --commission-rate 0.10 \
  --amount 10000000uumee \
  --pubkey $(umeed tendermint show-validator) \
  --chain-id canon-2 \
  --min-self-delegation "1" \
  --gas-prices 0.1uumee \
  --gas-adjustment 1.5 \
  --gas auto \
  --moniker "COSMØSTAKE" \
  --identity "xxxx" \
  --website "https://cosmostake.space" \
  --details "Our crypto community aspires to a decentralized future. Check Twitter https://twitter.com/COSM0STAKE" \
  --from xxxx
  ```
