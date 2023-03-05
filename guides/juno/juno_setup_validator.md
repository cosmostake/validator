# JUNO
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
git checkout v12.0.0
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
cp cosmovisor/cosmovisor $GOPATH/bin/cosmovisor
```
```console
cd $HOME
```

## Configure Cosmovisor

First, create the required directories:
```console
mkdir -p $HOME/.juno/cosmovisor/genesis/bin
```
```console
mkdir $HOME/.juno/cosmovisor/upgrades
```

Then copy the ``junod`` binanry to the ``genesis/bin`` folder:
```console
cp $GOPATH/bin/junod $HOME/.juno/cosmovisor/genesis/bin
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
9.0.0
```

This should print the version number of junod.

## Initialize and Configure Juno node

Now let's initialize the node:
```console
NODE_MONIKER=<YOUR_NODE_MONIKER>
```
```console
junod config chain-id juno-1
```
```console
junod init $NODE_MONIKER --chain-id juno-1
```

## Download the latest Genesis File

Download the latest genesis file:
```console
curl https://share.blockpane.com/juno/phoenix/genesis.json > $HOME/.juno/config/genesis.json
```

Check the genesis file:
```console
sha256sum $HOME/.juno/config/genesis.json
```

## Setup Seeds and Persistance Peers

First do:
```console
seeds=""
```
```console
peers="87ed42f2dd265013f3e5a6643ff6e0fffadb9aa0@juno.nodejumper.io:29656,531601a86419128f5d2752c13da8d378d3e84a4e@95.217.121.45:26656,5def3483df94aa27dc792f9f8741522ad6a43a2f@176.9.155.55:26656,72e877c245e22a0dd0a8fd2a74f0008081748d99@65.108.137.36:26656,eba3e4b8a82dbe8e3d2c2702b20e4e3b2f14a3a3@94.130.71.181:26856,4b0f1f25d5bff62d6cd674ddbe56df14d58979f3@141.94.253.173:46656,36c38f6be7ec9dc62e85f8d9880e007fda9bcd89@5.161.69.17:26656,21c772008019fe194b575cd2b69d1fa5ed2f421c@65.108.193.249:2070,c44a49da8adf6ab86a26c9b7fa53da179597605b@85.10.243.90:26656,4e1c2471efb89239fb04a4b75f9f87177fd91d00@95.217.82.72:26656,ca2533c240a384570e04cb2a166e16af587b8600@65.21.199.116:26656,76bb9d009212e42046b17f37bd92cf283eb74913@34.77.203.227:26656,9ac1907e6d6567054b13581aabf6fdf41f62946c@65.108.99.254:29656,ae758c7a08e49b7fddbda7460fd9c9f5b3f837ba@78.47.48.187:26656,b18cb0bb546274189d23b05409f81663a714ba58@65.109.22.163:16188,b227b38632fef30df5c92d38e60ad822f8534394@142.132.255.70:26656,2d48ab95a06c0284f706eacc2ac23e7cc19a271c@34.243.74.137:26656,524855d7ade67148a6d9f045553d3ceb4fc0c13b@5.9.62.249:26656,aa188da9d61e5a95ee28d7e5ae70bb12114a7916@157.90.35.145:27656,929bbfd1859a68a0e93392e952783ab8cdfbb91d@198.244.229.1:26656,f5a3bb76ad3fb61b0b78d3ccf79703c5292f45bb@3.250.8.64:26656,f90ab44ae1c38915a99d0c1605b64989d543ad72@51.81.208.145:26656,735c2d46aa3cb4109b6463b8bce43060ecb8887a@167.235.80.155:16656,82588f011491c6100d922d133f52fc23460b9231@95.217.91.232:26656,7abf03632ce48440b8f4a0c770b3806abf21b3bd@95.217.228.124:26656,8222761893d968ba0df4fc7e1677a66b532cb185@195.201.163.15:26656,3951e31ca7edad2869c1d160cf55822790444563@23.88.3.31:26656,2d23f1a5e5fd739d8406b209bd6b602465494e3a@167.235.80.154:26656,be1aef25d4c92b80daa0f9857a5e68fb3985dd7b@142.132.151.144:26656,4d93027e85fd815cef2459d76447b41d6930ee59@198.244.179.127:26656,b2c932d42a81c529328fe968eee387d7918faae2@162.55.232.44:26656,2ed6df7c98ca4ef9c40fcdce255daf56e3e502d5@51.81.208.3:26656,8be9603f3db2b14da70fe6c065f313d0157af8d0@35.237.0.24:26656,4d1468f230067003a9d4e6d7dd70f82311c5e304@65.108.139.112:26656,e7c642bdd79fd79cd2677f4f8b1351236b5ec2f3@204.16.241.208:26656,1bc41e3dcf63ff0246996b72a2d012798af0e57f@142.132.154.213:26656,f8c768174bf4842d64d823fb434b0847d71145c0@5.9.49.11:26656,e785ff41486bf5f217a043d404ccaaff85c0c804@51.79.228.120:26656,b3c80f4609607fb8ac9740e22786c600870f0a50@147.182.165.242:26656,d83892be2e6efc38e255943ce86ae8229d2aee90@178.128.220.188:26656,e069fb15eb31926b5e5f4a56287a207e8db82bf8@65.144.145.234:26656,514d070a29f307f5162da055b4fef3040274e8e4@65.109.25.157:26656,a3297ad0cb76b22fb1770762fbc81b6104371e77@34.73.18.249:26656,bfeae5219b310065f4785fddaa96aca69e66b0e6@141.98.219.163:26656,bba10290da32f3cb41e15c3a192413666ce05cee@5.9.208.8:26656,70fcee92283edc02340289b2a74e4ab1a0203848@162.55.244.250:39656,5036b8465ba1e37c122897be926ddafa445f64e6@77.220.141.51:26856,c8060b8b05cfb93a93a663a7127ab2ba7aec3db1@95.217.211.157:26656,386dcf9393093e0d93f46545898ca1b5bad78899@65.108.134.78:33095,055f2f6a1fa45a02f3e15cd4c13156083edb63af@147.75.92.19:46656,100083e611d5c93e9653869213c328fd69a97e06@89.149.218.191:26656"
```
Script for you to update ``persistent_peers`` and ``seeds`` setting with these peers in ``config.toml``
```console
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.juno/config/config.toml
```
Set `minimum-gas-prices`:
```console
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0025ujuno,0.001ibc\/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9"|g' $HOME/.juno/config/app.toml
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
Current version of snapshot always avaible [here](https://polkachu.com/tendermint_snapshots/juno)
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
wget -O juno_4942936.tar.lz4 https://snapshots1.polkachu.com/snapshots/juno/juno_4942936.tar.lz4 --inet4-only
```

Decompress the snapshot to your database location:
```console
lz4 -c -d juno_4942936.tar.lz4  | tar -x -C $HOME/.juno
```

Remove downloaded snapshot to free up space:
```console
rm -v juno_4942936.tar.lz4
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
junod keys add <wallet> --keyring-backend file
```

Recover Keplr acoount. Use your `mnemonic` phrase. 
Send token ``JUNO`` to it. 

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
