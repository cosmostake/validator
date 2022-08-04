# Sifchain
``Setup validator node manually``

## Update OS and Install Prerequisites

Update the local package list and install any available updates:
```console
sudo apt-get update && sudo apt upgrade -y
```
Install essential tools etc:
```console
sudo apt-get install make build-essential gcc git jq chrony mc htop curl -y
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

## Install Sifnode

Clone the Sifnode github repository:
```console
git clone https://github.com/Sifchain/sifnode.git
```

Go to ``sifnode`` directory:
```console
cd sifnode
```

Take new release:
```console
git checkout v0.13.5
```

Compile and install:
```console
make install
```
Check that the installation was successful:
```console
sifnoded version
```

This should print the version number of sifnoded.

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
git checkout v0.45.6
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
mkdir -p $HOME/.sifnoded/cosmovisor/genesis/bin
```
```console
mkdir $HOME/.sifnoded/cosmovisor/upgrades
```

Then copy the ``sifnoded`` binanry to the ``genesis/bin`` folder:
```console
cp $GOPATH/bin/sifnoded $HOME/.sifnoded/cosmovisor/genesis/bin
```
Now we need to add some environment variables that Cosmovisor is dependent on:
```console
export DAEMON_HOME=$HOME/.sifnoded
export DAEMON_RESTART_AFTER_UPGRADE=true
export DAEMON_ALLOW_DOWNLOAD_BINARIES=false
export DAEMON_NAME=sifnoded
export UNSAFE_SKIP_BACKUP=true
```

Check everything done:
```console
cosmovisor version
```

Output 
```
0.13.5
```

This should print the version number of sifnoded.

## Initialize and Configure Sifnode

Now let's initialize the node:
```console
sifnoded init coinside --chain-id sifchain-1
```

where "coinside" is the my moniker and "sichain-1" the chain id.

## Download the latest Genesis File

Go to the ``config`` directory:
```console
cd .sifnoded/config
```

Download the latest genesis file in this directory:
```console
wget -O genesis.json.gz https://raw.githubusercontent.com/Sifchain/networks/master/betanet/sifchain-1/genesis.json.gz
```

Unzip the genesis file:
```console
gunzip genesis.json.gz
```
## Setup Seeds and Persistance Peers

First do (Polkachu.com seed and peers):
```console
SEEDS=ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:32656
```
```console
PEERS=97f3fe2e8c6692d37d61ec7f2194a11e315664ee@65.108.248.146:26656,4cdfae6364e51832355143ab7dbfd17821414e80@65.108.0.197:26656,abdd98763fb0437f9c67e308e6ae14df8c690639@116.203.29.162:55656,1725f7fe10016f9eaec15bdc0e5e316200bfcc03@43.131.28.128:26656,7b554cbb9b260c950ef47f864a4be9176bd7c913@162.55.98.45:29504,553225a2d4b74c67351afaba55361164a8198ac0@192.81.130.203:26656,9c3b58b6bc28b5ccec66eb7b45b7980c0872729b@45.56.117.204:26656,30f2c8299d132d8b10b07b85da6a97271e61bfe0@178.63.44.171:26656,bd1ac99f62e83486800ceee2b056fcf4763294fe@65.108.135.84:26656,f3a7eedaef6bb87582d2073f53d18b24181e74cf@45.32.235.55:26656,3b833d53a12ee6ea0e3b89fa282f1a020a8ba01b@65.108.77.106:26759,79c33ecab7bd0a3a20f9479ee16004e49d154b0e@167.235.236.27:26656,eb5c90b88d18d1c20beed9581e2213f5306830bb@162.55.143.10:32656,bc073ddee70eda6a65ce9093b4c9da9f7cb78452@66.206.6.82:26665,300be3e950a7871d8c085511577d0fe0512599f8@157.90.36.57:26656,663dec65b754aceef5fcccb864048305208e7eb2@34.248.110.88:26656,26f483714389cdff71c97b8c4cfd04c069bda1f8@144.76.114.157:26656,2b9e00adb745979007ee499ab91c58805961e76c@65.21.238.100:36656,239baa0b73b33a1f8099c30a593539397e700240@62.171.143.207:26656,b77a57efb60599e7aa9f8dc74aea6a0eb68fff5e@45.77.123.145:26656,f33f1bb8894253daf9242cef96092686792cef80@161.97.157.60:26656,186be99c12835cc53de73452c839b2575de3cb03@66.29.156.21:26656,917ea24d568725b4df16f2b49c6d9e7b610befbe@95.217.122.107:26656,844bf674f03a98b8bb87d5ea3ac3cd5b76db64f3@23.88.52.117:26656,584dfa4fcf6ffb587371fc5c33ea834355b58486@35.162.197.129:26656,e72228cb0b6591e529cccb16019a70ca013a8387@78.46.75.70:26656,90326699211d820e3cfe972e82879a4939c275ad@146.19.24.36:26656,aa0efa6a2eef3b4f472012341728044fe691afeb@154.53.56.86:26656,e954ea53a2244cf4dc2a33b9284d324cc8422745@104.243.33.234:26656,c52a0851338c46e1e2cba01bd902e857c5f3cdf5@178.154.195.75:26656,e38678bc262964b8a9f363f4e5e921c5f405485f@65.108.137.38:26656,08e773600f34ad92c968b71c1fe5d9a4472b7ff1@66.94.121.251:26656,cb6ae22e1e89d029c55f2cb400b0caa19cbe5523@99.79.66.30:26603,bdc353d95b3c5b6861829a71bd1faa9b205bb4a4@142.132.209.228:36656,0120f0a48e7e81cc98829ef4f5b39480f11ecd5a@52.76.185.17:26656,6a95f61051cfdeca0bfe771c3373ee7f11e6df4b@35.74.104.174:26656,cf75747457723aafd0c5bb863547193fc395fde4@154.12.227.186:16656,bcc2d07a14a8a0b3aa202e9ac106dec0bef91fda@13.55.247.60:26656,6b37c01ae15298322273c3d7e4ec0c670962ac27@138.201.207.43:26656,80b95a9e87574aa80e352e94ba0405cbbb54145e@20.86.80.175:26656,cc3631d2c2efbb0fc5ed76466ffc40bc656bea4a@135.181.60.250:24656,409b338131be4bca7c508bedb63b8823071846cc@167.235.29.19:26627,0b9c998626ed7b2375f49762adcff30551207c6f@135.181.114.98:26656,a95b1775ebb0573afe4be02977d17572d4d25c33@142.132.152.187:10556,49f700aedf582a8444c3df2b6a8dba2b72fbff8e@50.116.10.88:26656,fcbac47c23284b82e7a26c2184eae822654b497c@65.108.137.39:26656,80151d310e0eedaa853ab769e9578fa8cfe571e6@178.128.241.82:26656,03ce0e30bae258a5ed4f7368b61b2744bdff6e3d@157.90.225.219:26656,a07fc82d6668bf79c5f42853ccf0b18b12194387@195.3.220.7:26656,acdd716c4cafb87e4a8fc9be79a199c15bd5732a@207.244.255.229:10073,9f5ebbbfc7bb73b30a15334af99a8b005d07b4de@185.37.145.2:26656,4a0099b47580cfbc5c8f7df9ab68559b433af5bd@5.9.137.186:26656,0b2674574624c1cf5c085d682afcace2ccc9f11c@5.9.137.98:26656,c2d7e035744aff2b30046b8aad4acc7d1034cd69@89.58.31.171:26627,653239811b08fb993f1b1073d2ce6f2252b3f81c@65.108.141.74:26656,c97f6b9766eebd7f60687aa6162057b089b63ec1@65.21.92.158:26656,eb3fcedd6625b52d86ccc16eac64a77372e9ad5a@3.16.114.170:26656,05277a5bb8aab8e92342880342b648025cdc34d8@51.91.67.184:30056,f62f88bd11b2b1c08247ac142b4f9f13e25fb861@46.33.97.20:26666,474496d6d12500a94e3dd4de219a5ebbcb286063@138.201.193.238:26656,0c3df294c63170e70b46b7759b194b60d4731af1@185.16.39.47:26656,7c38e97c188c25f2d407af7b3c4d6af80360527f@85.237.192.105:26656,0d4981bdaf4d5d73bad00af3b1fa9d699e4d3bc0@44.235.108.41:26656,ad1227c2fe92a3894825744a6810efae891432cf@176.191.97.120:26656,dca5b75b8697e96be0fbb5e259525c24d3d0e64b@142.132.152.204:26656,e6682a24696ec95b91a74907f1ca9d50782f6a7a@38.242.193.53:26656,476f7c5878ffe8dbfb8ee288ac99cac7df8c8e81@65.108.225.126:26656,a0a0f6af8488599aab6016778570937b12f8acb6@45.79.205.177:26676,a0ed1e46bce772071418e2adafb1488ac28352ff@168.119.109.71:30012,ca92d99cdf25d108d82f00557bbbe067b8511f0c@195.3.220.67:26656,38886c525b4d38b3526cebc1de4ab837919cfa20@46.226.128.217:28656,2ad81b7a7a28499bc8dfea18b01690836b95e140@83.243.40.90:26656,642f74d63741f6d49c8c28aae27c653fad6527cd@146.19.24.216:26656,7575e45f006f558f15d2dcd26b68de88d580dced@144.2.71.66:46665,e9ecc82a10392d16a0de4036a703697d42c49086@66.29.147.23:26656
```
Script for you to update ``persistent_peers`` and ``seeds`` setting with these peers in ``config.toml``
```console
sed -i "s/seeds =.*/seeds = \"$SEEDS\"/g" $HOME/.sifnoded/config/config.toml
```
```console
sed -i "s/persistent_peers =.*/persistent_peers = \"$PEERS\"/g" $HOME/.sifnoded/config/config.toml
```

Check. Don't be lazy:
```console
cat config.toml
```

## Setup the node's External Address

Next we need to set up the external address which is the public IP address of the node. Change the ``<bind_ip_address>`` to the address of your node:
```console
sed -i -e "s/external_address = \"\"/external_address = \"<bind_ip_address>:26656\"/g" $HOME/.sifnoded/config/config.toml
```

## Setup pruning

Polkachu.com settings:
```console
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.sifnoded/config/app.toml
```
```console
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.sifnoded/config/app.toml
```
```console
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.sifnoded/config/app.toml
```

Pruning:
```
pruning-keep-recent = "100"
pruning-keep-every = "0"
pruning-interval = "10"
```

## Setup Snapshot Polkachu.com

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
wget -O sifchain_7725913.tar.lz4 https://snapshots1.polkachu.com/snapshots/sifchain/sifchain_7725913.tar.lz4
```

Decompress the snapshot to your database location:
```console
lz4 -c -d sifchain_7725913.tar.lz4  | tar -x -C $HOME/.sifnoded
```

Remove downloaded snapshot to free up space:
```console
rm -v sifchain_7725913.tar.lz4
```

## Setup Service file

Use editor ``nano``:
```console
nano /etc/systemd/system/sifnoded.service
```

Output:
```
[Unit]
Description=cosmovisor
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/cosmovisor start --x-crisis-skip-assert-invariants
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=sifnoded"
Environment="DAEMON_HOME=/root/.sifnoded"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```
Let's try to start ``sifnoded``. First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```

Then enable the ``sifnoded`` service:
```console
sudo -S systemctl enable sifnoded
```

We can now start ``sifnoded``:
```console
sudo systemctl start sifnoded
```

We can check that the service is running:
```console
sudo systemctl status sifnoded
```

Check logs:
```console
journalctl -f -n 200 -u sifnoded
```

## Wallet preparation:

Let's generate a new key:
```console
# change <wallet> to yours
sifnoded keys add <wallet> --keyring-backend file
```

Recover Keplr acoount. Use your `mnemonic` phrase. 
Send token ``ROWAN`` to it. 

## Setup Validator

Make transaction. to change the ``moniker``, ``from`` and ``chain-id`` and other values as required:
```console
# change <wallet> to yours
sifnoded tx staking create-validator \
  --commission-max-change-rate 0.1 \
  --commission-max-rate 0.2 \
  --commission-rate 0.05 \
  --amount 100000000000000000rowan \
  --pubkey $(sifnoded tendermint show-validator) \
  --chain-id sifchain-1 \
  --min-self-delegation "1" \
  --gas 300000 \
  --fees 100000000000000000rowan \
  --moniker "COIN SIDE" \
  --identity=9B2EB22C7DAC8684 \
  --website="https://coin-side.com/nastavnichestvo" \
  --details="Crypto community who care about decentralized future" \
  --from <wallet> \
  --node tcp://rpc.sifchain.finance:80 \
  --keyring-backend file
  ```
  Unjail
  ```console
  sifnoded tx slashing unjail --from coinside --chain-id sifchain-1 --gas 300000
  ```
  
  Governance vote
  ```console
  sifnoded tx gov vote 98 yes --from coinside --chain-id sifchain-1 --fees 100000000000000000rowan --gas auto -y
  ```
