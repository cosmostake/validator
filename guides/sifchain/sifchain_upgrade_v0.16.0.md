# Sifchain v0.16.0 Upgrade

The Upgrade is scheduled for block `8311470`. A countdown clock is [here](https://www.mintscan.io/sifchain/blocks/8311470)

This guide assumes that you use cosmovisor to manage upgrades

```bash
# get the new version
cd sifnoded
git pull
git checkout v0.16.0
make install
```

# check the version

```bash
# should be 0.16.0
sifnoded version
# Should be commit fa9be3fb704fc8cce62053ea5ff70dc07ba3fae0
sifnoded version --long
```

# Make new directory and copy binary

```bash
mkdir -p $HOME/.sifnoded/cosmovisor/upgrades/0.16.0/bin
cp $HOME/go/bin/sifnoded $HOME/.sifnoded/cosmovisor/upgrades/0.16.0/bin
```

# check the version again

```bash
# should be 0.16.0
$HOME/.sifnoded/cosmovisor/upgrades/0.16.0/bin/sifnoded version
```
