# Sifchain 1.0.13-beta Upgrade

The Upgrade is scheduled for block `9063137`. A countdown clock is [here](https://www.mintscan.io/sifchain/blocks/9063137)

This guide assumes that you use cosmovisor to manage upgrades

```bash
# get the new version
cd sifchain
git pull
git checkout v1.0.13-beta
make install
```

# check the version

```bash
# should be 1.0.13-beta
sifnoded version
# Should be commit ea4cb0b02f1f6119fb30a06d24284b6705c438f1
sifnoded version --long
```

# Make new directory and copy binary

```bash
mkdir -p $HOME/.sifnoded/cosmovisor/upgrades/1.0.13-beta/bin
cp $HOME/go/bin/sifnoded $HOME/.sifnoded/cosmovisor/upgrades/1.0.13-beta/bin
```

# check the version again

```bash
# should be 1.0.13-beta
$HOME/.sifnoded/cosmovisor/upgrades/1.0.13-beta/bin/sifnoded version
