# Persistence Testnet
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
git clone https://github.com/persistenceOne/persistenceCore.git
```

Go to ``persistence`` directory:
```console
cd persistenceCore
```

Take new release:
```console
git checkout v7.0.0
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
mkdir -v -p $HOME/.persistenceCore/cosmovisor/genesis/bin
```
```console
mkdir -v $HOME/.persistenceCore/cosmovisor/upgrades
```

Then copy the ``persistenceCore`` binanry to the ``genesis/bin`` folder:
```console
cp -v $GOPATH/bin/persistenceCore $HOME/.persistenceCore/cosmovisor/genesis/bin
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
persistenceCore config chain-id test-core-1
```
```console
persistenceCore init $NODE_MONIKER --chain-id test-core-1
```

## Download the latest Genesis File

Download the latest genesis file:
```console
curl https://raw.githubusercontent.com/persistenceOne/networks/master/test-core-1/final_genesis.json > $HOME/.persistenceCore/config/genesis.json
```

Check the genesis file:
```console
sha256sum $HOME/.persistenceCore/config/genesis.json
```

## Setup Seeds and Persistance Peers

First do:
```console
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:15456"
```
```console
peers=560505f362da037bf4312f2a190744faf889ea54@125.143.190.194:13002,08bf766787b548f5b091ac51f8f96965452a4ee0@135.181.138.152:26656,ee239d2498bd073968e774887f813ac1fd4e50b3@95.216.242.183:26656,83eb077f79759d563dd765241ac83047e4bcbe8b@95.217.119.122:26656,a8f67a0287c4b4619a4c020b0dd0b40201a22229@65.21.77.175:19007,f517f91c2e7ea443ffc9c543ede16734f9000b5b@65.108.79.246:26703,0129a6ba9212455f0f69db6a6a0264490e454ecb@13.208.223.192:26656,c98c8d578b652ab5c2bff6273e390ba67a9ffde2@136.243.105.228:26656,6c3aa622558bb41c902f65f7a867e9aa684cffcc@49.12.14.205:26656,8de0a3e97190d24b75736a2db6ebc107aeaafce8@93.115.25.18:31656,71e39c43d87d1fe7c2d606c870599a497be52bdb@43.131.18.234:26656,4822400ec10c1ee84d37e93277ef7832c5426510@83.171.249.107:26656,723218672704e92a65100ddc28cd5719ada07686@3.24.177.45:26656,5e2a636207ff7028474863f6b7c52bcb8b64810d@89.117.63.227:26656,c01bb5af14b5ec609bd62ea1edb8218a731e2712@5.9.121.118:26656,e46e42065d8fb108b8f2add2539b16b01f0544f4@13.244.233.149:26656,4c07ba5a50acf901a3c0f51a93d37e3dc2c6b8c5@51.79.178.76:26656,097ee46a16f1198a05b6b96c71c1eb91943022a7@94.250.202.235:26756,14ecdc5126ea8d93c7d3a863d9d38e380e46fc06@185.225.233.30:26656,5c2a752c9b1952dbed075c56c600c3a79b58c395@195.3.220.54:26896,9ae8fc7430d9ba58b1982af1fdbc1a94647aec1b@65.108.131.99:21337,f0463724a5eee7f6b4df6ca49f79b0deee465c1a@144.126.135.137:26556,b9e897c3a9a59ec41866162761042b338bfbab35@65.109.81.119:19656,1ec7628853408df8269ba4c93b82f2cda2274cdd@95.165.149.94:24656,4871fc0e8897a91f1ea074b86f13a82a1236d068@222.106.187.14:53900,9022710bdbe336b9539e6dd257f621b58d18cab2@162.62.63.58:26656,286917df41767a84a19a0777c4d1bc3b6523c4ad@16.170.226.142:26656,61c6448536ca4445465c2fa907a5e7ebb688b2a4@65.21.202.61:19007,f14ebf4b13f7bca0daaa973444bfe87fdc8c969e@24.207.176.67:26756,80b8e4ba55a5bd943e02143cd42a45d47e088598@167.235.115.119:26656,30859d0fb31dd08894ba15ac90bf020b26ba5890@23.88.53.28:56666,0a1841abd75d4f31106ef0a91fb094192f9423a5@54.180.152.136:26656,4d623afb4ea704de83f29158a9ceccfc21730d70@94.130.132.227:2130,784148b50d4444f0527f9e12659f2057f8a3eb7d@65.108.225.158:15456
```
Script for you to update ``persistent_peers`` and ``seeds`` setting with these peers in ``config.toml``
```console
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.persistenceCore/config/config.toml
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
fresh snapshot you can always find [here](https://polkachu.com/testnets/persistence/snapshots)

Download the snapshot:
```console
sudo apt install snapd -y
```
```console
sudo snap install lz4
```
```console
wget -O persistence_10714064.tar.lz4 https://snapshots.polkachu.com/testnet-snapshots/persistence/persistence_10714064.tar.lz4 --inet4-only
```
```console
rm -r ~/.persistenceCore/wasm
```

Decompress the snapshot to your database location:
```console
lz4 -c -d persistence_10714064.tar.lz4  | tar -x -C $HOME/.persistenceCore
```

Remove downloaded snapshot to free up space:
```console
rm -v persistence_10714064.tar.lz4
```

## Setup Service file

Use editor ``nano``:
```console
nano /etc/systemd/system/persistenceCore.service
```

Output:
```
[Unit]
Description=Persistence Node Testnet
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
  --chain-id test-core-1 \
  --min-self-delegation "1" \
  --moniker <moniker> \
  --identity <keybase> \
  --website <website> \
  --details <details> \
  --from <wallet>
  ```
Unjail:
  ```console
  persistenceCore tx slashing unjail --from <wallet> --chain-id test-core-1
  ```
  
Governance vote:
  ```console
  persistenceCore tx gov vote 10 yes --from <wallet> --chain-id test-core-1
  ```

Check block height:
```console
curl -s localhost:20657/status | jq .result.sync_info.latest_block_height
```

Withdraw Commission And Rewards From Your Validator
```console
persistenceCore tx distribution withdraw-rewards $(persistenceCore keys show <wallet> --bech val -a) --commission --from <wallet> --chain-id test-core-1 -y
```

Delegate to yourself (Self bonded)
```console
persistenceCore tx staking delegate $(persistenceCore keys show <wallet> --bech val -a) 1000000uxprt --from <wallet> --chain-id test-core-1 -y
```
