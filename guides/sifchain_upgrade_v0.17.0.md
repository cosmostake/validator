# Sifchain v0.17.0 Upgrade

The Upgrade is scheduled for block `8367070`. A countdown clock is [here](https://www.mintscan.io/sifchain/blocks/8367070)

This guide assumes that you use cosmovisor to manage upgrades

```bash
# get the new version
cd sifnoded
git pull
git checkout v0.17.0
make install
```

# check the version

```bash
# should be 0.17.0
sifnoded version
# Should be commit a20c79b58026f025493e5e52deaed5dc0ad50f02
sifnoded version --long
```

# Make new directory and copy binary

```bash
mkdir -p $HOME/.sifnoded/cosmovisor/upgrades/0.17.0/bin
cp $HOME/go/bin/sifnoded $HOME/.sifnoded/cosmovisor/upgrades/0.17.0/bin
```

# check the version again

```bash
# should be 0.17.0
$HOME/.sifnoded/cosmovisor/upgrades/0.17.0/bin/sifnoded version
```
