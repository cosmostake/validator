# install dependencies, if needed
sudo apt update
sudo apt install -y curl git jq lz4 build-essential

if [ ! -f "/usr/local/go/bin/go" ]; then
  bash <(curl -s "https://raw.githubusercontent.com/nodejumper-org/cosmos-scripts/master/utils/go_install.sh")
  source .bash_profile
fi

NODE_MONIKER=<YOUR_NODE_MONIKER>

cd || return
rm -rf osmosis
git clone https://github.com/osmosis-labs/osmosis
cd osmosis || return
git checkout v11.0.1
make install
osmosisd version # v11.0.1

osmosisd config chain-id osmosis-1
osmosisd init $NODE_MONIKER --chain-id osmosis-1

curl https://github.com/osmosis-labs/networks/raw/main/osmosis-1/genesis.json > $HOME/.osmosisd/config/genesis.json
sha256sum $HOME/.osmosisd/config/genesis.json # 1cdb76087fabcca7709fc563b44b5de98aaf297eedc8805aa2884999e6bab06d

sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001uosmo"|g' $HOME/.osmosisd/config/app.toml
seeds="21d7539792ee2e0d650b199bf742c56ae0cf499e@162.55.132.230:2000,295b417f995073d09ff4c6c141bd138a7f7b5922@65.21.141.212:2000,ec4d3571bf709ab78df61716e47b5ac03d077a1a@65.108.43.26:2000,4cb8e1e089bdf44741b32638591944dc15b7cce3@65.108.73.18:2000,f515a8599b40f0e84dfad935ba414674ab11a668@osmosis.blockpane.com:26656,6bcdbcfd5d2c6ba58460f10dbcfde58278212833@osmosis.artifact-staking.io:26656"
peers="83c06bc290b6dffe05aa9cec720bedfc118afcbc@osmosis.nodejumper.io:35656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.osmosisd/config/config.toml

# in case of pruning
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.osmosisd/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.osmosisd/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "10"|g' $HOME/.osmosisd/config/app.toml
sed -i 's/snapshot-interval *=.*/snapshot-interval = 0/g' $HOME/.osmosisd/config/app.toml

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
