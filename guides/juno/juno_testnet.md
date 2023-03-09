# JUNO testnet
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
## Install JUNO

Clone the Juno github repository:
```console
git clone https://github.com/CosmosContracts/juno
```

Go to ``juno`` directory:
```console
cd juno
```

Take new release:
```console
git checkout v13.0.0
```

Compile and install:
```console
make install
```
Check that the installation was successful:
```console
junod version
```

This should print the version number of junod.

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
git checkout v0.45.12
```

Compile and install:
```console
make cosmovisor
```

Then copy the ``cosmovisor`` binanry to the ``genesis/bin`` folder:
```console
cp -v cosmovisor/cosmovisor $GOPATH/bin/cosmovisor
```
```console
cd $HOME
```

## Configure Cosmovisor

First, create the required directories:
```console
mkdir -p -v $HOME/.juno/cosmovisor/genesis/bin
```
```console
mkdir -v $HOME/.juno/cosmovisor/upgrades
```

Then copy the ``junod`` binanry to the ``genesis/bin`` folder:
```console
cp -v $GOPATH/bin/junod $HOME/.juno/cosmovisor/genesis/bin
```
Now we need to add some environment variables that Cosmovisor is dependent on:
```console
export DAEMON_HOME=$HOME/.juno
export DAEMON_RESTART_AFTER_UPGRADE=true
export DAEMON_ALLOW_DOWNLOAD_BINARIES=false
export DAEMON_NAME=junod
export UNSAFE_SKIP_BACKUP=true
```

Check everything done:
```console
cosmovisor version
```

Output 
```
12.0.0
```

This should print the version number of junod.

## Initialize and Configure Juno node

Now let's initialize the node:
```console
NODE_MONIKER=<YOUR_NODE_MONIKER>
```
```console
junod init $NODE_MONIKER --chain-id uni-6
```

## Download the latest Genesis File

Download the latest genesis file:
```console
curl https://raw.githubusercontent.com/CosmosContracts/testnets/main/uni-6/genesis.json > ~/.juno/config/genesis.json
```

Check the genesis file:
```console
sha256sum $HOME/.juno/config/genesis.json
```

## Setup Seeds and Persistance Peers

First do:
```console
seeds="ba41ec3f89892e1b490d94354453bee80b8b53b6@49.12.176.137:26656,c54bf418fb542634495f57a1e36c9bd057d55e1b@5.161.80.115:26656,5adba84a219227d3cf6c2de163d49d6765538a3b@162.19.61.144:26656,c478980dee1acc6416874434772eb063acdc6821@135.181.59.162:12656,9db06fae1998a14c79cb13d50152828b9fa049e9@195.201.161.122:26656,93effaf34cc23a18abf3ebef61404c6eb11aaf59@173.249.12.165:26656,256a94965d448b56bd05e01eb4af082f5e65cfc2@222.106.187.14:53706,5479526dbdf4f27aa59ddc52be9cf2614049d28e@65.21.199.148:26798"
```

Script for you to update ``seeds`` setting with these peers in ``config.toml``
```console
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|' $HOME/.juno/config/config.toml
```
Set `minimum-gas-prices`:
```console
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025ujunox\"/" ~/.juno/config/app.toml
```
Addrbook:
```console
wget -O addrbook.json https://snapshots.polkachu.com/testnet-addrbook/juno/addrbook.json --inet4-only
```
```console
mv addrbook.json ~/.juno/config
```
Check. Don't be lazy:
```console
cd .juno/config && cat config.toml
```

## Setup pruning

Settings:
```console
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.juno/config/app.toml
```
```console
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.juno/config/app.toml
```
```console
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.juno/config/app.toml
```

Pruning:
```
pruning-keep-recent = "100"
pruning-keep-every = "0"
pruning-interval = "10"
```

## Setup Snapshot Polkachu.com
Current version of snapshot always avaible [here](https://polkachu.com/testnets/juno/snapshots)
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
wget -O juno_451092.tar.lz4 https://snapshots.polkachu.com/testnet-snapshots/juno/juno_451092.tar.lz4 --inet4-only
```

Decompress the snapshot to your database location:
```console
lz4 -c -d juno_451092.tar.lz4  | tar -x -C $HOME/.juno
```

Remove downloaded snapshot to free up space:
```console
rm -v juno_451092.tar.lz4
```

## Setup Service file

Use editor ``nano``:
```console
nano /etc/systemd/system/junod.service
```

Output:
```
[Unit]
Description=Juno Node
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=junod"
Environment="DAEMON_HOME=/root/.juno"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```

Let's try to start ``junod``. First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```

Then enable the ``junod`` service:
```console
sudo -S systemctl enable junod
```

We can now start ``junod``:
```console
sudo systemctl start junod
```

We can check that the service is running:
```console
sudo systemctl status junod
```

Check logs:
```console
journalctl -f -n 200 -u junod
```

## Wallet preparation:

Let's generate a new key:
```console
# change <wallet> to yours
junod keys add <wallet>
```

Recover Keplr acoount. Use your `mnemonic` phrase. 
Send token ``JUNOX`` to it. 
Testnet tokens can be requested from the `#faucet` channel on Discord.
To request tokens type `$request <your-public-address>` in the message field and press enter.

## Setup Validator

Make transaction. to change the ``moniker``, ``from`` and ``chain-id`` and other values as required:
```console
# change <wallet> to yours
junod tx staking create-validator \
  --commission-max-change-rate 0.01 \
  --commission-max-rate 0.20 \
  --commission-rate 0.05 \
  --amount 1000000ujuno \
  --pubkey $(junod tendermint show-validator) \
  --chain-id juno-1 \
  --min-self-delegation "1" \
  --gas-prices 0.1ujuno \
  --gas-adjustment 1.5 \
  --gas auto \
  --moniker "COIN SIDE" \
  --identity <your_keybase> \
  --website="https://github.com/COIN-SIDE/validator" \
  --details="Our crypto community aspires to a decentralized future" \
  --from <wallet>
  ```
Unjail
  ```console
  junod tx slashing unjail --from <wallet> --chain-id juno-1 --gas-prices 0.1ujuno --gas-adjustment 1.5 --gas auto -y
  ```
  
  Governance vote
  ```console
  junod tx gov vote 30 yes --from <wallet> --chain-id juno-1 --gas-prices 0.1ujuno --gas-adjustment 1.5 --gas auto -y
  ```
