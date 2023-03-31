Server: 1 CPU / 2 GB RAM / 5 GB SSD
# Installation the light node
1) Preparing the server
```
sudo apt update && sudo apt upgrade -y && sudo apt-get install -y build-essential curl wget jq
```
2) Installs Go:
```
cd $HOME
wget https://golang.org/dl/go1.19.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.5.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source ~/.bash_profile
```

3) Check Go version: 
```
go version
```
4) Download the binary file:
```
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node/ 
git checkout tags/v0.8.0 
make build 
make install 
make cel-key
```
5) Check Celestia version:
```
celestia version
```
```
Semantic version: v0.8.0 
Commit: ef582655342c73384a66314972428b152227e428 
Build Date: Thu Dec 15 10:19:22 PM UTC 2022 
System version: amd64/linux 
Golang version: go1.19.1 
```

6) Create key for node
Create a new key
```
./cel-key add your-key --keyring-backend test --node.type light --p2p.network blockspacerace
```
Restore an existing key using a mnemonic (optional)
```
./cel-key add your-key --keyring-backend test --node.type light --p2p.network blockspacerace --recover
```
Then we can go to **#faucet** channel for “Blockspace Race” and request test tokens in format:
```
$request celestia1rt3...
```
7) Initialize Light node
```
celestia light init \
  --keyring.accname your-key \
  --p2p.network blockspacerace
  ```
  
8) Create a service file:
```
sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-lightd.service
[Unit]
Description=celestia-lightd Light Node
After=network-online.target
 
[Service]
User=$USER
ExecStart=/usr/local/bin/celestia light start --core.ip https://rpc-blockspacerace.pops.one/ --core.grpc.port 9090 --gateway --gateway.addr localhost --gateway.port 26659 --p2p.network blockspacerace
Restart=on-failure
RestartSec=3
LimitNOFILE=4096
 
[Install]
WantedBy=multi-user.target
EOF
```
9) Start the node:
```
sudo systemctl enable celestia-lightd
sudo systemctl start celestia-lightd && sudo journalctl -u celestia-lightd.service -f
```
Check wallet
```
cd celestia-node
./cel-key list --node.type light --keyring-backend test --p2p.network blockspacerace
```
Get light node ID:
```
AUTH_TOKEN=$(celestia light auth admin --p2p.network blockspacerace)
curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
 ```    
Check the logs:
```
sudo journalctl -u celestia-lightd.service -f
```
