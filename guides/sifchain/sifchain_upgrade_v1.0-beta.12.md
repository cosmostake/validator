# Sifchain v1.0-beta.12 Upgrade

The Upgrade is scheduled for block `8786836`. A countdown clock is [here](https://www.mintscan.io/sifchain/blocks/8786836)

This guide assumes that you use cosmovisor to manage upgrades

```bash
# get the new version
cd sifchain
git pull
git checkout v1.0-beta.12
make install
```

# check the version

```bash
# should be 1.0-beta.12
sifnoded version
# Should be commit 50554216551af7d823d211ab850e61cdfa62fd7f
sifnoded version --long
```

# Make new directory and copy binary

```bash
mkdir -p $HOME/.sifnoded/cosmovisor/upgrades/1.0-beta.12/bin
cp $HOME/go/bin/sifnoded $HOME/.sifnoded/cosmovisor/upgrades/1.0-beta.12/bin
```

# check the version again

```bash
# should be 1.0-beta.12
$HOME/.sifnoded/cosmovisor/upgrades/1.0-beta.12/bin/sifnoded version
```
