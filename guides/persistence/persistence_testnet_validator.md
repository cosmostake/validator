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
git checkout v8.1.0
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
persistenceCore config chain-id test-core-2
```
```console
persistenceCore init $NODE_MONIKER --chain-id test-core-2
```

## Download the latest Genesis File

Download the latest genesis file:
```console
curl https://raw.githubusercontent.com/persistenceOne/networks/master/test-core-2/genesis.json > $HOME/.persistenceCore/config/genesis.json
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
peers=762c339b3f7626b9ebcfe7286f1e133eec587164@65.108.98.235:58556,f6ef633ef1caa621f3f4bea7a7ef31463fc9a43e@95.216.184.59:26656,40f63c75b6afe61e74af428cc669eb481089a587@65.108.225.158:15456,8eb88b7edf912ac9d63b183b42823f220a3fc226@65.108.192.123:48656,9551e08e22f13ee6dc769731cd131d59f9731332@65.108.79.246:26706,98f6fc6560ebec5a1d2099ecf91213b53575ec81@65.21.77.175:11006,8525f72c1adf9cf9b7dc8deb08f904a71acaea36@5.161.203.187:26656,18ba18bbcc1bdee0ac703bc0cd143fae3599aa29@95.165.149.94:24656,2dd629fc08c97061c64f15f934c1c368565870d6@217.76.53.18:36656,ac80febd6313b39e70d3e5ed50afddc491d79258@195.189.96.111:31656,7f971fc5fc2ffedbaf32f3b4021645571461a712@198.244.177.67:26656,b2f036bfa207249363ea2ac935e6bed3d4c10dbd@37.252.184.239:26656,cee6b94965f301e8b5ad905a65fa39c03cd193ce@51.68.152.17:26656,c3e785f853677ad3677542f9e2679a20957ca104@217.76.61.200:26756,7fd0511997417324ab46ac85a935d89998d1e4e2@94.130.76.179:26656,21ca0b996db604681fb73721ecb01d2c6410c628@162.19.94.46:26656,0134c0a888ab07d67c4a41afcb103b76266fb0e7@65.21.202.61:11006,82e69659bef9c385b4934727f43d69a673f6f033@5.161.191.128:26656,fc86e4edef908edc6699bbecdf568cc7743d8d07@125.143.190.194:13002,591ceb144a142275ac3e51ed310aa1b6eb04e20c@217.76.48.63:38556,30859d0fb31dd08894ba15ac90bf020b26ba5890@23.88.53.28:56666,ee239d2498bd073968e774887f813ac1fd4e50b3@95.216.242.183:26656,8de0a3e97190d24b75736a2db6ebc107aeaafce8@93.115.25.18:31656,3ce04530d850e727d092e9a81d9f580c7ff8db56@141.94.97.77:26656,8a9bdbdb6fefc6929187093b4a4994350ec47ad4@195.14.6.2:26656,2585af35660dcfb0a02dd8bc2e766d11f5603f99@73.117.148.143:26656,75739f2e430a1ff8e9224cf9702f4a6d8c16a3db@5.161.212.155:26656,60a79ab50eda4ab65ea071a76a84bbf5949f1000@165.22.25.27:26656,20b4f9207cdc9d0310399f848f057621f7251846@222.106.187.13:20800,9b341b2b1d6d453a79c911ad7b2d032e117184cf@136.243.105.228:26656,2aa052d4b05525f3329ad3f32793e5983bf48bab@139.59.13.194:26656

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
wget -O persistence_1481929.tar.lz4 https://snapshots.polkachu.com/testnet-snapshots/persistence/persistence_1481929.tar.lz4 --inet4-only
```
```console
rm -r ~/.persistenceCore/wasm
```

Decompress the snapshot to your database location:
```console
lz4 -c -d persistence_1481929.tar.lz4  | tar -x -C $HOME/.persistenceCore
```

Remove downloaded snapshot to free up space:
```console
rm -v persistence_1481929.tar.lz4
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
  --chain-id test-core-2 \
  --min-self-delegation "1" \
  --moniker <moniker> \
  --identity <keybase> \
  --website <website> \
  --details <details> \
  --from <wallet> \
  --gas-prices 0.005uxprt \
  --gas-adjustment 1.5
  ```
Unjail:
  ```console
  persistenceCore tx slashing unjail --from <wallet> --chain-id test-core-2
  ```
  
Governance vote:
  ```console
  persistenceCore tx gov vote 10 yes --from <wallet> --chain-id test-core-2
  ```

Check block height:
```console
curl -s localhost:20657/status | jq .result.sync_info.latest_block_height
```

Withdraw Commission And Rewards From Your Validator
```console
persistenceCore tx distribution withdraw-rewards $(persistenceCore keys show <wallet> --bech val -a) --commission --from <wallet> --chain-id test-core-2 -y
```

Delegate to yourself (Self bonded)
```console
persistenceCore tx staking delegate $(persistenceCore keys show <wallet> --bech val -a) 1000000uxprt --from <wallet> --chain-id test-core-2 -y
```
