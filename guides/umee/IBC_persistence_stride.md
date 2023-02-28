# IBC relayer Persistence <> Stride
``Setup relayer manually``

Don't forget to check `go` version first:
```console
go version
```

## Clone the latest release ([releases page](https://github.com/cosmos/relayer/releases)):
```console
git clone https://github.com/cosmos/relayer.git
```
```console
cd relayer
```

Checkout:
```console
git checkout v2.2.0
```

Install:
```console
make install
```

Check:
```console
rly version
```
The output should be like:
```
version: 2.2.0
commit: 98abd889c8c0fcff366d1e77409593f2ee03499c
cosmos-sdk: v0.46.8
go: go1.19.5 linux/amd64
```

## Initialize the relayer
```console
rly config init --memo <your_memo>
```

## Configure the Chains to Relay Between
```console
rly chains add stride persistence
```

## Create Keys
```console
rly keys add persistence <wallet>
```
```console
rly keys add stride <wallet1>
```

## Edit the Relayer Key Name in `config.yaml`
```console
nano ~/.relayer/config/config.yaml
```

## Ensure the Keys are Funded
```console
rly query balance persistence
```
```console
rly query balance stride
```

## Configure Path Metadata
```console
rly paths fetch
```

## Check
```console
rly chains list
```
Output should show all checkboxes:
```
1: core-1               -> type(cosmos) key(✔) bal(✔) path(✔)
2: stride-1             -> type(cosmos) key(✔) bal(✔) path(✔)
```

## Configure the Channel Filter
By default, the relayer will relay packets over all channels on a given connection. You can find all open IBC channels between Persistence and other chains [here](https://www.mintscan.io/persistence/relayers).

How should `~/.relayer/config/config.yaml` look:
```
global:
    api-listen-addr: :5183
    timeout: 10s
    memo: cosmostake
    light-cache-size: 20
chains:
    persistence:
        type: cosmos
        value:
            key: cosmostake
            chain-id: core-1
            rpc-addr: https://rpc-persistence.architectnodes.com:443
            account-prefix: persistence
            keyring-backend: test
            gas-adjustment: 1.2
            gas-prices: 0.01uxprt
            min-gas-amount: 0
            debug: false
            timeout: 20s
            output-format: json
            sign-mode: direct
            extra-codecs: []
    stride:
        type: cosmos
        value:
            key: cosmostake1
            chain-id: stride-1
            rpc-addr: https://stride.nodejumper.io:443
            account-prefix: stride
            keyring-backend: test
            gas-adjustment: 1.2
            gas-prices: 0.01ustrd
            min-gas-amount: 0
            debug: false
            timeout: 20s
            output-format: json
            sign-mode: direct
            extra-codecs: []
paths:
    persistence-stride:
        src:
            chain-id: core-1
            client-id: 07-tendermint-83
            connection-id: connection-71
        dst:
            chain-id: stride-1
            client-id: 07-tendermint-54
            connection-id: connection-33
        src-channel-filter:
            rule: allowlist
            channel-list: [channel-67]
 ```
 ## Create Service
 ```console
 nano /etc/systemd/system/ibc.service
 ```
 Put:
 ```
[Unit]
Description=IBC
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/rly start --memo <your_memo>
Restart=always
RestartSec=3
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
```
First reload the systemctl daemon:
```console
sudo -S systemctl daemon-reload
```
Then enable the `ibc` service:
```console
sudo -S systemctl enable ibc
```

 ## Start Relayer
 We can now start `ibc`:
```console
sudo systemctl start ibc
```
Check logs:
```console
journalctl -f -n 200 -u ibc
```
 
