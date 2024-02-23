---
cover: >-
  https://images.unsplash.com/flagged/photo-1554386690-8627e1041100?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwxMHx8Yml0Y29pbnxlbnwwfHx8fDE3MDU4NjM1OTN8MA&ixlib=rb-4.0.3&q=85
coverY: 0
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Bitcoin Firehose Baremetal Guide

## Bitcoin Core

### Pre-requisites

{% code fullWidth="true" %}
```bash
mkdir -p /mnt/data
cd /mnt/
sudo chown -R payne:payne data
cd data
mkdir -p bitcoin/core
mkdir github
cd github
wget <https://bitcoincore.org/bin/bitcoin-core-26.0/bitcoin-26.0-x86_64-linux-gnu.tar.gz>
dtrx bitcoin-26.0-x86_64-linux-gnu.tar.gz
rm -rf bitcoin-26.0-x86_64-linux-gnu.tar.gz
cd bitcoin-26.0-x86_64-linux-gnu
mv bitcoin-26.0 ../
cd ..
rm -rf bitcoin-26.0-x86_64-linux-gnu
```
{% endcode %}

### Permissions

{% code fullWidth="true" %}
```bash
mkdir -p /mnt/data/bitcoin/core
sudo useradd -r -s /bin/false bitcoin
cd /mnt/data
sudo chown -R bitcoin:bitcoin /mnt/data/bitcoin/core
sudo chown -R bitcoin:bitcoin /mnt/data/github/bitcoin-26.0
sudo chmod -R 755 /mnt/data
```
{% endcode %}

### Systemd service file

{% code fullWidth="true" %}
```bash
# It is not recommended to modify this file in-place, because it will
# be overwritten during package upgrades. If you want to add further
# options or overwrite existing ones then use
# $ systemctl edit bitcoind.service
# See "man systemd.service" for details.

# Note that almost all daemon options could be specified in
# /etc/bitcoin/bitcoin.conf, but keep in mind those explicitly
# specified as arguments in ExecStart= will override those in the
# config file.

[Unit]
Description=Bitcoin daemon
Documentation=https://github.com/bitcoin/bitcoin/blob/master/doc/init.md

# <https://www.freedesktop.org/wiki/Software/systemd/NetworkTarget/>
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/mnt/data/github/bitcoin-26.0/bin/bitcoind -pid=/mnt/data/github/bitcoin-26.0/bin/bitcoind.pid \\
                                                     -conf=/mnt/data/github/bitcoin-26.0/bitcoin.conf \\
                                                     -datadir=/mnt/data/bitcoin/core \\
                                                     -startupnotify='systemd-notify --ready' \\
                                                     -shutdownnotify='systemd-notify --stopping' \\
                                                     -rpcbind="0.0.0.0" \\
                                                     -rpcport=8332 \\
                                                     -rpcallowip="0.0.0.0/0" \\
                                                     -rpcauth=bitcoin:64b10798ddf4a7711ba80abd37e69560$0eab8edf372bc6a62abc57ebc17500118750f4632377189cce572c35342d64e2 \\
                                                     -server \\
                                                     -disablewallet

# Process management
####################

Type=notify
NotifyAccess=all
PIDFile=/mnt/data/github/bitcoin-26.0/bin/bitcoind.pid

Restart=on-failure
TimeoutStartSec=infinity
TimeoutStopSec=600

# Directory creation and permissions
####################################

# Run as bitcoin:bitcoin
User=bitcoin
Group=bitcoin

# /run/bitcoind
RuntimeDirectory=bitcoind
RuntimeDirectoryMode=0710

# /etc/bitcoin
ConfigurationDirectory=bitcoin
ConfigurationDirectoryMode=0710

# /var/lib/bitcoind
StateDirectory=bitcoind
StateDirectoryMode=0710

# Hardening measures
####################

# Provide a private /tmp and /var/tmp.
PrivateTmp=true

# Mount /usr, /boot/ and /etc read-only for the process.
ProtectSystem=full

# Deny access to /home, /root and /run/user
ProtectHome=true

# Disallow the process and all of its children to gain
# new privileges through execve().
NoNewPrivileges=true

# Use a new /dev namespace only populated with API pseudo devices
# such as /dev/null, /dev/zero and /dev/random.
PrivateDevices=true

# Deny the creation of writable and executable memory mappings.
MemoryDenyWriteExecute=true

[Install]
WantedBy=multi-user.target
```
{% endcode %}

### Optional: Generate your own rpc auth:

{% code overflow="wrap" fullWidth="true" %}
```bash
wget <https://raw.githubusercontent.com/bitcoin/bitcoin/master/share/rpcauth/rpcauth.py>
chmod +x rpcauth.py
./rpcauth <user> <password>

# copy the -rpcauth to your systemd file
```
{% endcode %}

### Start Bitcoin Core

{% code fullWidth="true" %}
```bash
sudo systemctl daemon-reload
sudo systemctl restart bitcoind
```
{% endcode %}

### Viewing logs

{% code fullWidth="true" %}
```bash
journalctl -fu bitcoind
```
{% endcode %}

## Bitcoin Firehose

### Pre-requisites

{% code overflow="wrap" fullWidth="true" %}
```bash

# Add to PATH
echo "export PATH="$PATH:/usr/local/go/bin:/root/.local/bin"" >> ~/.zshrc
source ~/.zshrc

# Install go
wget <https://go.dev/dl/go1.21.2.linux-amd64.tar.gz> && rm -rf /usr/local/go && tar -C /usr/local -xzf go1.21.2.linux-amd64.tar.gz && rm go1.21.2.linux-amd64.tar.gz
source ~/.zshrc
go version
```
{% endcode %}

### Building Firehose

{% code overflow="wrap" fullWidth="true" %}
```bash
cd /mnt/data/github
git clone <https://github.com/streamingfast/firehose-bitcoin> # no release tag for now
git clone -b v1.0.0 <https://github.com/streamingfast/firehose-core>

cd /mnt/data/github/firehose-bitcoin
go install -v ./cmd/firebtc
cd /mnt/data/github/firehose-core
go install -v ./cmd/firecore

# binary builds in your GOPATH: ~/go/bin

rm -rf /mnt/data/github/firehose-core
rm -rf /mnt/data/github/firehose-bitcoin

mkdir -p /mnt/data/github/firehose/bin
cp ~/go/bin/firecore /mnt/data/github/firehose/bin/
cp ~/go/bin/firebtc /mnt/data/github/firehose/bin/
```
{% endcode %}

### Permissions

{% code fullWidth="true" %}
```bash
sudo useradd -r -s /bin/false firehose
sudo chown -R bitcoin:bitcoin /mnt/data/bitcoin/firehose
sudo chown -R bitcoin:bitcoin /mnt/data/github/firehose
sudo chmod -R 755 /mnt/data
```
{% endcode %}

### Systemd service file

{% code overflow="wrap" fullWidth="true" %}
```bash
[Unit]
Description=SF Firehose
StartLimitIntervalSec=200
StartLimitBurst=5

[Service]
Type=simple
User=firehose
Group=firehose
SyslogIdentifier=firehose
ExecStart=/mnt/data/github/firehose/bin/firecore -c /mnt/data/github/firehose/config.yml start
Restart=on-failure
LimitNOFILE=1000000
RestartSec=30
TimeoutStopSec=30m

[Install]
WantedBy=multi-user.target
```
{% endcode %}

### Generate salted login:

{% code overflow="wrap" fullWidth="true" %}
```yaml
echo -n 'bitcoin:bitcoin' | base64 

# replace with your desired username
# if you replace it, you'll need to generate a new rpcauth string and replace the existing one inside the bitcoind.service file
```
{% endcode %}

### Config YML file

{% code overflow="wrap" fullWidth="true" %}
```yaml
start:
  args:
    - merger
    - relayer
    - firehose
    - substreams-tier1
    - substreams-tier2
    - reader-node
  flags:
####################### GENERAL #################################
    data-dir: /mnt/data/bitcoin/firehose
####################### LOGGING ################################
    log-format: stackdriver
    log-to-file: false
##################### READER NODE ###############################
    reader-node-debug-firehose-logs: false
    reader-node-grpc-listen-addr: :9000
    reader-node-manager-api-addr: :8080
    reader-node-blocks-chan-capacity: 5000
    reader-node-readiness-max-latency: 1200s
    reader-node-working-dir: /mnt/data/bitcoin/firehose
    reader-node-data-dir: /mnt/data/bitcoin/firehose
    reader-node-path: /mnt/data/github/firehose/bin/firebtc
    reader-node-arguments: "poller 0 --reader-state-storage-path=/mnt/data/bitcoin/firehose --rpc-endpoint=http://127.0.0.1:8332 --headers='Authorization: Basic Yml0Y29pbjpiaXRjb2lu'"
```
{% endcode %}

### Start Firehose

{% code overflow="wrap" fullWidth="true" %}
```bash
sudo systemctl daemon-reload
sudo systemctl restart firehose
```
{% endcode %}

### Viewing logs

{% code overflow="wrap" fullWidth="true" %}
```bash
journalctl -fu firehose
```
{% endcode %}
