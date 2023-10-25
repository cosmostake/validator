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
git checkout v6.1.0
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
umeed init $NODE_MONIKER --chain-id canon-4
```

## Download the latest Genesis File

Download the latest genesis file:
```console
wget https://canon-4.rpc.network.umee.cc/genesis
mv -v $HOME/genesis $HOME/.umee/config/genesis.json
```
Remove ``{"jsonrpc":"2.0","id":-1,"result":{"genesis":`` from the start and ``}}`` from the end to create the genesis file.

## Setup Peers

```console
val-1 = ee7d691781717cbd1bf6f965dc45aad19c7af05f@canon-4.network.umee.cc:10000
val-2 = dfd1d83b668ff2e59dc1d601a4990d1bd95044ba@canon-4.network.umee.cc:10001
val-3 = e25008728d8f800383561d5ce68cff2d6bfc3826@canon-4.network.umee.cc:10002
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
# change <wallet_pfd> to yours
umeed keys add <wallet_pfd>
```

Recover Keplr acoount. Use your `mnemonic` phrase. 
Faucet Canon-4 https://faucet.umee.cc/

## Install price-feeder
```console
cd $HOME/umee/price-feeder
```
```console
git checkout price-feeder/umee/v2.1.7
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
gas_adjustment = 1.03

[server]
listen_addr = "0.0.0.0:7171"
read_timeout = "20s"
verbose_cors = true
write_timeout = "20s"

[[deviation_thresholds]]
base = "MARS"
threshold = "1.5"

[[deviation_thresholds]]
base = "CMST"
threshold = "1.5"

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
base = "BTC"
threshold = "2"

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

[[deviation_thresholds]]
base = "AKT"
threshold = "1.5"

[[deviation_thresholds]]
base = "LUNA"
threshold = "1.5"

[[deviation_thresholds]]
base = "WAXL"
threshold = "1.5"

[[deviation_thresholds]]
base = "MATIC"
threshold = "1.5"

[[deviation_thresholds]]
base = "DOT"
threshold = "1.5"

[[deviation_thresholds]]
base = "stkATOM"
threshold = "1.5"

[[deviation_thresholds]]
base = "qATOM"
threshold = "1.5"

[[deviation_thresholds]]
base = "CBETH"
threshold = "1.5"

[[deviation_thresholds]]
base = "stJUNO"
threshold = "2"

[[deviation_thresholds]]
base = "USK"
threshold = "2"

[[deviation_thresholds]]
base = "stUMEE"
threshold = "2"

[[deviation_thresholds]]
base = "RETH"
threshold = "2"

[[deviation_thresholds]]
base = "LINK"
threshold = "2"

[[deviation_thresholds]]
base = "WETH"
threshold = "2"

[[deviation_thresholds]]
base = "WBTC"
threshold = "2"

[[deviation_thresholds]]
base = "NCT"
threshold = "2"

[[deviation_thresholds]]
base = "SOMM"
threshold = "2"

[[currency_pairs]]
base = "UMEE"
providers = [
  "gate",
]
quote = "USDT"

[[currency_pairs]]
base = "UMEE"
providers = [
  "osmosis",
]
quote = "ATOM"

[[currency_pairs]]
base = "USDT"
providers = [
  "kraken",
  "coinbase",
  "crypto",
]
quote = "USD"

[[currency_pairs]]
base = "USDC"
providers = [
  "kraken",
]
quote = "USD"

[[currency_pairs]]
base = "ATOM"
providers = [
  "bitget",
]
quote = "USDT"

[[currency_pairs]]
base = "ATOM"
providers = [
  "kraken",
  "crypto",
]
quote = "USD"

[[currency_pairs]]
base = "USDC"
providers = [
  "bitget",
  "kraken",
]
quote = "USDT"

[[currency_pairs]]
base = "DAI"
providers = [
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
  "bitget",
  "crypto",
]
quote = "USDT"

[[currency_pairs]]
base = "CRO"
providers = [
  "crypto",
  "bitget",
]
quote = "USDT"

[[currency_pairs]]
base = "BNB"
providers = [
  "mexc",
  "bitget",
]
quote = "USDT"

[[currency_pairs]]
base = "OSMO"
providers = [
  "osmosis",
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
base = "WETH"
providers = [
  "eth-uniswap"
]

quote = "USDC"
[[currency_pairs.pair_address_providers]]
address = "0x88e6a0c2ddd26feeb64f039a2c41296fcb3f5640"
provider = "eth-uniswap"

[[currency_pairs]]
base = "WBTC"
providers = [
  "eth-uniswap"
]

quote = "WETH"
[[currency_pairs.pair_address_providers]]
address = "0xcbcdf9626bc03e24f779434178a73a0b4bad62ed"
provider = "eth-uniswap"

[[currency_pairs]]
base = "CBETH"
providers = [
  "eth-uniswap"
]

quote = "WETH"
[[currency_pairs.pair_address_providers]]
address = "0x840deeef2f115cf50da625f7368c24af6fe74410"
provider = "eth-uniswap"

[[currency_pairs]]
base = "LINK"
providers = [
  "eth-uniswap"
]

quote = "WETH"
[[currency_pairs.pair_address_providers]]
address = "0xa6cc3c2531fdaa6ae1a3ca84c2855806728693e8"
provider = "eth-uniswap"

[[currency_pairs]]
base = "stATOM"
providers = [
  "osmosis",
]
quote = "ATOM"

[[currency_pairs]]
base = "stOSMO"
providers = [
  "osmosis",
]
quote = "OSMO"

[[currency_pairs]]
base = "IST"
providers = [
  "osmosis",
]
quote = "OSMO"

[[currency_pairs]]
base = "IST"
providers = [
  "crescent",
]
quote = "USDC"

[[currency_pairs]]
base = "USDC"
providers = [
  "kraken",
]
quote = "USD"

[[currency_pairs]]
base = "AKT"
quote = "USDT"
providers = [ "huobi", "gate",]

[[currency_pairs]]
base = "AKT"
quote = "USD"
providers = [ "kraken", "crypto",]

[[currency_pairs]]
base = "JUNO"
quote = "USDT"
providers = [ "bitget", "mexc",]

[[currency_pairs]]
base = "JUNO"
quote = "USD"
providers = [ "kraken",]

[[currency_pairs]]
base = "JUNO"
quote = "ATOM"
providers = [ "osmosis",]

[[currency_pairs]]
base = "LUNA"
quote = "USDT"
providers = [ "gate", "huobi", "bitget",]

[[currency_pairs]]
base = "WAXL"
providers = [
  "kraken",
]
quote = "USD"

[[currency_pairs]]
base = "WAXL"
providers = [
  "huobi",
  "bitget",
  "gate",
]
quote = "USDT"

[[currency_pairs]]
base = "DOT"
providers = [
  "kraken",
  "coinbase",
  "crypto",
]
quote = "USD"

[[currency_pairs]]
base = "DOT"
providers = [
  "gate",
  "bitget",
]
quote = "USDT"

[[currency_pairs]]
base = "MATIC"
providers = [
  "coinbase",
  "kraken",
]
quote = "USD"

[[currency_pairs]]
base = "MATIC"
providers = [
  "gate",
  "mexc",
  "bitget",
]
quote = "USDT"

[[currency_pairs]]
base = "stkATOM"
providers = [
  "osmosis",
]
quote = "ATOM"

[[currency_pairs]]
base = "qATOM"
providers = [
  "osmosis",
]
quote = "ATOM"

[[currency_pairs]]
base = "CBETH"
quote = "USD"
providers = [ "coinbase",]

[[currency_pairs]]
base = "CMST"
providers = [
  "osmosis",
]
quote = "OSMO"

[[currency_pairs]]
base = "MARS"
providers = [
  "osmosis",
]
quote = "OSMO"

[[currency_pairs]]
base = "stJUNO"
providers = [
  "osmosis",
]
quote = "JUNO"

[[currency_pairs]]
base = "USK"
providers = [
  "kujira",
]
quote = "USDC"

[[currency_pairs]]
base = "stUMEE"
providers = [
  "osmosis",
]
quote = "UMEE"

[[currency_pairs]]
base = "RETH"
providers = [
  "eth-uniswap",
]

quote = "WETH"
[[currency_pairs.pair_address_providers]]
address = "0xa4e0faA58465A2D369aa21B3e42d43374c6F9613"
provider = "eth-uniswap"


[[currency_pairs]]
base = "NCT"
providers = [
  "osmosis",
]
quote = "OSMO"

[[currency_pairs]]
base = "SOMM"
providers = [
  "osmosis",
]
quote = "OSMO"

[[currency_pairs]]
base = "SOMM"
providers = [
  "gate",
]
quote = "USDT"

[account]
address = "umee1..."
chain_id = "canon-4"
validator = "umeevaloper1"

[keyring]
backend = "os"
dir = "/home/.umee"
pass = ""

[rpc]
grpc_endpoint = "localhost:9090"
rpc_timeout = "100ms"
tmrpc_endpoint = "http://localhost:26657"

[telemetry]
enable-hostname = true
enable-hostname-label = true
enable-service-label = true
enabled = true
global-labels = [["chain_id", "canon-4"]]
service-name = "price-feeder"
prometheus-retention-time = 100

[[provider_endpoints]]
name = "binance"
rest = "https://api1.binance.com"
websocket = "stream.binance.com:9443"

[[provider_endpoints]]
name = "osmosis"
rest = "https://api.osmo-api.prod.ojo.network"
websocket = "api.osmo-api.prod.ojo.network"

[[provider_endpoints]]
name = "crescent"
rest = "https://api.cresc-api.prod.ojo.network"
websocket = "api.cresc-api.prod.ojo.network"

[[provider_endpoints]]
name = "eth-uniswap"
rest = "http://104.197.233.185:8000/subgraphs/name/ojo-network/unidexer"
websocket = "not supported"
```

Create service:
```console
nano /etc/systemd/system/price-feeder.service
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

Let's try to start ``price-feeder``. First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```

Then enable the ``price-feeder`` service:
```console
sudo -S systemctl enable price-feeder
```

We can now start ``price-feeder``:
```console
sudo systemctl start price-feeder
```

We can check that the service is running:
```console
sudo systemctl status price-feeder
```

Check logs:
```console
journalctl -f -n 200 -u price-feeder
```

## Delegate price-feeder address
```console
umeed tx oracle delegate-feed-consent <wallet_addr> <wallet_pfd_addr> --fees 30000uumee --chain-id canon-4
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
  --chain-id canon-4 \
  --min-self-delegation "1" \
  --gas-prices 0.1uumee \
  --gas-adjustment 1.5 \
  --gas auto \
  --moniker <moniker> \
  --identity <keybase> \
  --website <website> \
  --details <details> \
  --from <wallet>
  ```
