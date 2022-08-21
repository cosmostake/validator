# Sifchain v0.15.0 Upgrade

The Upgrade is scheduled for block `8170210`. A countdown clock is [here](https://www.mintscan.io/sifchain/blocks/8170210)

This guide assumes that you use cosmovisor to manage upgrades

```bash
# get the new version
cd sifnoded
git pull
git checkout v0.15.0
make install
```

# check the version

```bash
# should be 0.15.0
sifnoded version
# Should be commit 560a8262b627e920546e27864759c1027f07d7b9
sifnoded version --long
```

# Make new directory and copy binary

```bash
mkdir -p $HOME/.sifnoded/cosmovisor/upgrades/0.15.0/bin
cp $HOME/go/bin/sifnoded $HOME/.sifnoded/cosmovisor/upgrades/0.15.0/bin
```

# check the version again

```bash
# should be 0.15.0
$HOME/.sifnoded/cosmovisor/upgrades/0.15.0/bin/sifnoded version
```
