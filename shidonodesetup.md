# Shido Node Setup Guide

Welcome to the Shido Node Setup Guide! This document will walk you through the necessary steps to install and configure your Shido node efficiently. Follow the instructions carefully to ensure a smooth setup.

## System Requirements

Ensure your system meets the following specifications for optimal performance:

- **CPU:** Minimum of 4 physical cores (8 cores recommended)
- **Storage:** At least 200GB NVMe SSD (700GB+ NVMe recommended for better I/O performance)
- **Memory:** Minimum 8GB RAM (16GB recommended)
- **Network:** At least 100Mbps bandwidth

## Prerequisites Installation

Begin by updating your package lists and installing the necessary dependencies:

```
sudo apt update
sudo apt install -y curl git jq lz4 wget unzip build-essential
```

## Install Go
Remove any existing Go installation and install the required version:

```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
```

# Node Installation
## Clone the Repository

Start by cloning the Shido node repository:
```
git clone https://github.com/ShidoGlobal/shidoupgrade_v2.0.0.git
```
## Binary Setup for Ubuntu
Choose the appropriate binary based on your Ubuntu version.

## For Ubuntu 20.04

Copy the binary to the local binaries directory and make it executable:
```
sudo cp shidoupgrade_v2.0.0/ubuntu20.04build/shidod /usr/local/bin/
sudo chmod +x /usr/local/bin/shidod
```
## For Ubuntu 22.04

Similarly, use the binary suited for Ubuntu 22.04:
```
sudo cp shidoupgrade_v2.0.0/ubuntu22.04build/shidod /usr/local/bin/
sudo chmod +x /usr/local/bin/shidod
```
## Library Download
Download the necessary library and move it to the appropriate directory:
```
wget -O libwasmvm.x86_64.so https://snapshots.256x25.tech/snapshots/shido/libwasmvm.x86_64.so --inet4-only
sudo mv libwasmvm.x86_64.so /usr/lib
```
## Node Initialization
Initialize your Shido node with a chosen name and chain ID:
```
shidod init YourNodeName --chain-id shido_9008-1
```
## Genesis File Setup
Retrieve and place the genesis file in the configuration directory:
```
wget -O genesis.json https://snapshots.256x25.tech/snapshots/shido/genesis.json --inet4-only
sudo mv genesis.json ~/.shidod/config
```
## CLI Configuration

Set the chain ID and keyring backend for the node CLI:
```
shidod config chain-id shido_9008-1
shidod config keyring-backend file
```
# Configuration Tweaks
Modify various configuration settings for optimal performance.

## Adjust timeout_commit
Change the commit timeout to enhance block processing speed:
```
sed -i 's/timeout_commit = "3s"/timeout_commit = "1s"/g' "$HOME/.shidod/config/config.toml"
```
## Set Minimum Gas Price
Define the minimum gas price required for transactions:
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.25shido"|' $HOME/.shidod/config/app.toml
```
## Update Listen Address
Allow the node to listen on all network interfaces:
```
sed -i -e "s%tcp://127.0.0.1:26657%tcp://0.0.0.0:26657%" $HOME/.shidod/config/config.toml
```
## Configure Peers
Set up persistent peers to maintain network connectivity:
```
PEERS=7ed728831ff441d18a8556b64afcaebc31b68c74@3.76.57.158:26656,f28f693053306fba8bf59c4a54b7bd9f89de7ebb@18.193.227.128:26656,181fcc5672fee87751eb369491744e85ba0651f5@18.153.233.126:26656,8d46e292347951d651486611abac77825a0c83f8@18.199.25.117:26656,cdf19a7234ee8ec12519f6ad066408f09e1b73e0@15.157.50.94:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.shidod/config/config.toml
```
# Shido Snapshot Server Configuration
## Install Snapd and LZ4
Ensure that snapd and lz4 are installed for snapshot handling:
```
sudo apt update
sudo apt install snapd -y
sudo snap install lz4
```
## Pruning Settings
Customize pruning settings to manage database size:
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
  $HOME/.shidod/config/app.toml
```
## Disable Indexer
Turn off the indexer to reduce resource usage:
```
sed -i \
  -e 's|^indexer *=.*|indexer = "null"|' \
  $HOME/.shidod/config/config.toml
```
## Snapshot Download and Extraction
Fetch the latest snapshot and extract it to your node’s data directory:
```
wget -O shido_snapshot.tar.lz4 https://snapshots.256x25.tech/snapshots/shido/shido_snapshot.tar.lz4
lz4 -c -d shido_snapshot.tar.lz4 | tar -x -C $HOME/.shidod
rm -v shido_snapshot.tar.lz4
```

# Service Setup
Create a systemd service to manage the Shido node:
```
sudo tee /etc/systemd/system/shidod.service > /dev/null << EOF
[Unit]
Description=Shido Node Service
After=network-online.target

[Service]
User=$USER
ExecStart=/usr/local/bin/shidod start
Restart=on-failure
RestartSec=10
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
## Starting the Node
Enable and start the Shido service:
```
sudo systemctl daemon-reload
sudo systemctl enable shidod.service
sudo systemctl start shidod
```
## Monitoring Logs
To view the real-time logs of your Shido node, use the following command:
```
sudo journalctl -u shidod -f --no-hostname -o cat
```

# Credits
A special thanks to the 256x25 Validator for providing the comprehensive guide and snapshots that made this setup possible. Visit 256x25.tech for more information and resources.

If you run into any issues or have questions, feel free to open an issue in the repository.
