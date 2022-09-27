# Osmosis v12.0.0 Upgrade

The upgrade is scheduled for block `6246000`. A countdown clock is [here](https://www.mintscan.io/osmosis/blocks/6246000)

This guide assumes that you use cosmovisor to manage upgrades

```bash
# get the new version
cd osmosis
git pull
git checkout v12.0.0
make install
```

# check the version

```bash
# should be 12.0.0
osmosisd version
# Should be commit aa727b5bf25019d561da5d6f82dd9a3892fda2be
osmosisd version --long
```

# Copy binary

```bash
mkdir -p $HOME/.osmosisd/cosmovisor/upgrades/v12/bin
cp $HOME/go/bin/osmosisd $HOME/.osmosisd/cosmovisor/upgrades/v12/bin
```

# check the version again

```bash
# should be 12.0.0
$HOME/.osmosisd/cosmovisor/upgrades/v12/bin/osmosisd version
```
