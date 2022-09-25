# Test IPFS collab cluster

- The VM-shard drive size have `80G`, the space allocated to IPFS is `65G`
- The ports opened `4001`(for IPFS comm),`9096`(for cluster comm, we will change this when in production)
- The follow nodes could work on low-memory devices (`1GB RAM` or less)
- The current `IPFS`, `IPFS CLUSTER`, `GO`  are working on the latest versions.
- After installing `GO`, `IPFS`, `IPFS cluster`, the cluster-follow will download the configuration file here (service.json): 
```console
ipfs-cluster-follow collab_cluster_sifchain init http://127.0.0.1:8080/ipfs/QmNNh12Mb6rgDS4Qk5CPuZ5ht9e7jaSK4ebJufjrpAyN4R
```
- With the same machine you can folow multiple clusters.

##	Check and install updates:

```console
sudo apt-get update && sudo apt upgrade -y
```

###	Install some essential tools:

```console
sudo apt-get install make  gcc git -y
```

###	Install Go:

```console
wget https://go.dev/dl/go1.19.1.linux-amd64.tar.gz
```
```console
tar -xvf go1.19.1.linux-amd64.tar.gz
```
```console
mv go /usr/local
```

###	Create Path for Go and set environment variables:

```console
mkdir $HOME/gopath
```

###	Edit .bashrc file:

```console
nano $HOME/.bashrc
```

### Insert the content to the end of the .bashrc file:

```
export GOROOT=/usr/local/go
export GOPATH=$HOME/gopath
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

### Update .bashrc file and check Go version:

```console
source ~/.bashrc
```
```console
go version
```

##	Install IPFS:

```console
wget https://dist.ipfs.tech/kubo/v0.15.0/kubo_v0.15.0_linux-amd64.tar.gz
```
```console
tar -xvzf kubo_v0.15.0_linux-amd64.tar.gz
```
```console
cd kubo
```
```console
sudo bash install.sh
```
```console
ipfs init --profile server
```
```console
ipfs --version
```
###	Change storage capacity for IPFS
```console
nano /root/.ipfs/config
```
to
```
"StorageMax": "65GB"
```

##	Run IPFS daemon as a service in the background:

```console
sudo nano /etc/systemd/system/ipfs.service
```

### Add to it the following:
```
[Unit]
Description=IPFS Daemon
After=syslog.target network.target remote-fs.target nss-lookup.target
[Service]
Type=simple
ExecStart=/usr/local/bin/ipfs daemon --enable-namesys-pubsub
User=root
LimitNOFILE=4096
Restart=always
RestartSec=3
[Install]
WantedBy=multi-user.target
```
###	Save and close the file and apply the new service

```console
systemctl daemon-reload
```
```console
systemctl enable ipfs
```
```console
systemctl start ipfs
```
```console
systemctl status ipfs
```

`Reboot your system and check that IPFS daemon is still active and running`


## Deploying IPFS-Cluster:

```console
git clone https://github.com/ipfs/ipfs-cluster.git $GOPATH/src/github.com/ipfs/ipfs-cluster
```
```console
cd $GOPATH/src/github.com/ipfs/ipfs-cluster
```
```console
make install
```

### Check successful installation by running:

```console
ipfs-cluster-service --version
```
```console
ipfs-cluster-ctl --version
```

### Init and Start cluster follow node :

```console
ipfs-cluster-follow collab_cluster_sifchain init http://127.0.0.1:8080/ipfs/QmNNh12Mb6rgDS4Qk5CPuZ5ht9e7jaSK4ebJufjrpAyN4R
```
```console
ipfs-cluster-follow collab_cluster_sifchain run
```

### Run IPFS-Cluster-follow as a service:

```console
nano /etc/systemd/system/ipfs-cluster.service
```
	
And insert to it:

```
[Unit]
Description=IPFS-Cluster Daemon
Requires=ipfs
After=syslog.target network.target remote-fs.target nss-lookup.target ipfs
[Service]
Type=simple
ExecStart=/root/gopath/bin/ipfs-cluster-follow collab_cluster_sifchain run
User=root
LimitNOFILE=4096
Restart=always
RestartSec=3
[Install]
WantedBy=multi-user.target
```

### Apply new service and run it:

```console
systemctl daemon-reload
```
```console
systemctl enable ipfs-cluster
```
```console
systemctl start ipfs-cluster
```
```console
systemctl status ipfs-cluster
```

`Reboot your machine and check that both IPFS and IPFS-Cluster-follow services are running`

```console
ipfs-cluster-follow collab_cluster_sifchain status
```

You will get an output like this:

```
Config folder: /root/.ipfs-cluster-follow/collab_cluster_sifchain
Config source URL: Available (http://127.0.0.1:8080/ipfs/QmNNh12Mb6rgDS4Qk5CPuZ5ht9e7jaSK4ebJufjrpAyN4R)
Cluster Peer online: true
IPFS peer online: true
Additional help
```

To visualise the pinned files on the follow node:

```console
ipfs-cluster-follow collab_cluster_sifchain list
```

by Discord: `love life#5310`
