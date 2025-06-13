
# ✅ Zero Gravity (0G) RPC Node Setup Guide — GCP Optimized

This guide will help you set up a **Zero Gravity (0G) RPC Node** on a **Google Cloud Platform (GCP)** VM (Ubuntu 20.04+), using simple step-by-step instructions.

---

## 🚀 System Requirements (GCP VM Recommended)

- **OS**: Ubuntu 20.04 LTS (or 22.04)
- **RAM**: 64 GB
- **CPU**: 8 Cores
- **Disk**: 1 TB SSD
- **Public IP**: Required

---

## 🔓 GCP Firewall: Open Required Ports

Name	zero-gravity-firewall

Direction	Ingress

Action on match	Allow

Targets	All instances in the network

Source IP Ranges	0.0.0.0/0

Ports
tcp:26656,26657,6060,1317,9090,9091,22

---

## 🧱 Step-by-Step Setup

### 1. Update System & Install Dependencies
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git jq build-essential gcc unzip wget lz4 -y
```

---

### 2. Install Go (v1.21.3)
```bash
cd $HOME
wget https://golang.org/dl/go1.21.3.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.21.3.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

---

### 3. Install 0G Binary
```bash
wget https://rpc-zero-gravity-testnet.bit-panda.rest/evmosd -O ~/0gchaind
chmod +x ~/0gchaind
sudo mv ~/0gchaind /usr/local/bin/0gchaind
0gchaind version
```

---

### 4. Setup Environment Variables
```bash
echo 'export MONIKER="my-node"' >> ~/.bashrc
echo 'export CHAIN_ID="zgtendermint_9000-1"' >> ~/.bashrc
echo 'export WALLET_NAME="wallet"' >> ~/.bashrc
echo 'export RPC_PORT="26657"' >> ~/.bashrc
source ~/.bashrc
```

---

### 5. Init Node & Download Genesis
```bash
0gchaind init $MONIKER --chain-id $CHAIN_ID
0gchaind config chain-id $CHAIN_ID
0gchaind config keyring-backend os
0gchaind config node tcp://localhost:$RPC_PORT
wget https://rpc-zero-gravity-testnet.bit-panda.rest/genesis.json -O ~/.0gchain/config/genesis.json
```

---

### 6. Add Peers
```bash
PEERS=$(curl -s https://rpc-zero-gravity-testnet.trusted-point.com/peers.txt)
sed -i "s/^persistent_peers =.*/persistent_peers = \"$PEERS\"/" ~/.0gchain/config/config.toml
```

---

### 7. Configure App Settings
```bash
# Pruning
sed -i.bak -e "s/^pruning =.*/pruning = \"custom\"/" ~/.0gchain/config/app.toml
sed -i -e "s/^pruning-keep-recent =.*/pruning-keep-recent = \"100\"/" ~/.0gchain/config/app.toml
sed -i -e "s/^pruning-interval =.*/pruning-interval = \"10\"/" ~/.0gchain/config/app.toml

# Gas Price & Indexer
sed -i "s/^minimum-gas-prices =.*/minimum-gas-prices = \"0ua0gi\"/" ~/.0gchain/config/app.toml
sed -i "s/^indexer =.*/indexer = \"kv\"/" ~/.0gchain/config/config.toml
```

---

### 8. Create systemd Service (Auto Restart)
```bash
sudo tee /etc/systemd/system/0gd.service > /dev/null <<EOF
[Unit]
Description=0G RPC Node
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/0gchaind start --home /root/.0gchain
Restart=always
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable 0gd
sudo systemctl start 0gd
```

---

### 9. Check Sync Status
```bash
0gchaind status | jq .sync_info
```
When `catching_up = false`, node is fully synced ✅

---

## ⚡ Bonus: Fast Sync with Snapshot (Optional)
```bash
sudo systemctl stop 0gd
cp ~/.0gchain/data/priv_validator_state.json ~/.0gchain/priv_validator_state.json.backup
0gchaind tendermint unsafe-reset-all --home $HOME/.0gchain --keep-addr-book
curl -L https://rpc-zero-gravity-testnet.trusted-point.com/latest_snapshot.tar.lz4 | lz4 -d | tar -xf - -C $HOME/.0gchain
mv ~/.0gchain/priv_validator_state.json.backup ~/.0gchain/data/priv_validator_state.json
sudo systemctl start 0gd
```

---

## 🎉 You're Done!
Your RPC node is live on Zero Gravity!

Want to run a validator too? Let me know!
