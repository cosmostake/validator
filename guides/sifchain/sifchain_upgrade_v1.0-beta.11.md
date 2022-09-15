# Sifchain v1.0-beta.11 Upgrade

The Upgrade is scheduled for block `8618770`. A countdown clock is [here](https://www.mintscan.io/sifchain/blocks/8618770)

This guide assumes that you use cosmovisor to manage upgrades

```bash
# get the new version
cd sifchain
git pull
git checkout v1.0-beta.11
make install
```

# check the version

```bash
# should be 1.0-beta.11
sifnoded version
# Should be commit 1e2f00104b63d725d1ab8e243ce322b8e0872196
sifnoded version --long
```

# Make new directory and copy binary

```bash
mkdir -p $HOME/.sifnoded/cosmovisor/upgrades/1.0-beta.11/bin
cp $HOME/go/bin/sifnoded $HOME/.sifnoded/cosmovisor/upgrades/1.0-beta.11/bin
```

# check the version again

```bash
# should be 1.0-beta.11
$HOME/.sifnoded/cosmovisor/upgrades/1.0-beta.11/bin/sifnoded version
```
