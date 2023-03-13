# RPC, gRPC and API REST

## Caddy

Create new directory:
```console
mkdir -v ~/caddy
```
```console
cd ~/caddy
```
Download latest version:
```console
wget https://github.com/caddyserver/caddy/releases/download/v2.6.4/caddy_2.6.4_linux_amd64.tar.gz
```
```console
tar -vxf caddy_2.6.4_linux_amd64.tar.gz
```
```console
mv -v caddy /usr/bin/
```
Test that it's working:
```console
caddy version
```
## Configure the Caddyfile

```console
nano ~/Caddyfile
```
Modify, and paste in everything below:
```
{
    servers {
  protocols h1 h2c
    }
}

(cors) {
  @cors_preflight method OPTIONS
  @cors header Origin {args.0}

  handle @cors_preflight {
    header Access-Control-Allow-Origin "{args.0}"
    header Access-Control-Allow-Methods "GET, POST, PUT, PATCH, DELETE"
    header Access-Control-Allow-Headers "Content-Type"
    header Access-Control-Max-Age "3600"
    respond "" 204
  }

  handle @cors {
    header Access-Control-Allow-Origin "{args.0}"
    header Access-Control-Expose-Headers "Link"
  }
}

# RPC
wss://umee-testnet-rpc.cosmostake.space {
    handle {
  reverse_proxy localhost:26657
    }
}
ws://umee-testnet-rpc.cosmostake.space {
    redir wss://umee-testnet-rpc.cosmostake.space{uri} permanent
}

# API REST
umee-testnet-rest.cosmostake.space {
    header Access-Control-Allow-Origin *
    header Access-Control-Allow-Methods "POST, GET, OPTIONS"
    @options {
  method OPTIONS
    }
    import cors https://umee-testnet-rest.cosmostake.space
    reverse_proxy localhost:1317
}

http://umee-testnet-rest.cosmostake.space {
    redir https://umee-testnet-rest.cosmostake.space{uri} permanent
}

# gRPC
http://umee-testnet-grpc.cosmostake.space {
    log {
  output file /var/log/grpc_access.log
    }
    reverse_proxy h2c://localhost:9090
}
```

## Check

Now you can check if you configured correctly, with:
```console
caddy fmt --overwrite
```
```console
caddy validate ~/Caddyfile
```
Which should return:
```
...
Valid configuration
```
You can now run caddy with:
```console
caddy run --config /root/Caddyfile
```
Which should return something like:
```
...
... [INFO] [<your.cool.url>] The server validated our request
... [INFO] [<your.cool.url>] acme: Validations succeeded; requesting certificates
... [INFO] [<your.cool.url>] Server responded with a certificate.
... [INFO][<your.cool.url>] Certificate obtained successfully
... [INFO][<your.cool.url>] Obtain: Releasing lock
```

## Run caddy as a service

```console
nano /etc/systemd/system/caddy.service
```
Example file below:
```
[Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target

[Service]
User=root
ExecStart=/usr/bin/caddy run --config /root/Caddyfile
ExecReload=/usr/bin/caddy reload --config /root/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```
`Save and exit. Close caddy if it's still running`

To have caddy start automatically at reboot:
```console
systemctl enable caddy
```
```console
systemctl start caddy
```
```console
systemctl status caddy
```
Check logs:
```console
journalctl -f -n 200 -u caddy
```
If you want to edit your Caddfile, edit it, then run:
```console
caddy reload
```

## Done


