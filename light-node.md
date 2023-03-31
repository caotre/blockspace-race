# Phần cứng
Cấu hình phần cứng yêu cầu tối thiểu:
>Memory: 2 GB RAM. CPU: Single Core. Disk: 5 GB SSD Storage. Bandwidth: 56 Kbps for Download/56 Kbps for Upload
# Cài đặt light node
1) Cài các thành phần cần thiết
```
sudo apt update && sudo apt upgrade -y && sudo apt-get install -y build-essential curl wget jq
```
2) Cài Go:
```
cd $HOME
wget https://golang.org/dl/go1.19.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.5.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source ~/.bash_profile
```

3) Kiểm tra phiên bản Go: 
```
go version
```

4) Tải binary Celestia node:
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
5) Kiểm tra phiên bản Celestia:
```
celestia version
```
```
Semantic version: v0.8.0 
Commit: ef582655342c73384a66314972428b152227e428 
Build Date: Thu Dec 15 10:19:22 PM UTC 2022 
System version: amd64/linux 
Golang version: go1.19.5 
```

6) Tạo key cho node
```
./cel-key add your-key --keyring-backend test --node.type light --p2p.network blockspacerace
```
Khôi phục một key đã tạo trước bằng nemonic (optional)
```
./cel-key add your-key --keyring-backend test --node.type light --p2p.network blockspacerace --recover
```
Vào Discord kênh **#faucet** lấy token faucet theo format:
```
$request celestia1rt3...
```
7) Initialize Light node
```
celestia light init \
  --keyring.accname your-key \
  --p2p.network blockspacerace
  ```
  
8) Tạo service file:
```
sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-lightd.service
[Unit]
Description=celestia-lightd Light Node
After=network-online.target
 
[Service]
User=$USER
ExecStart=/usr/local/bin/celestia light start --core.ip https://rpc-blockspacerace.pops.one --core.rpc.port 26657 --core.grpc.port 9090 --keyring.accname my_celes_key --metrics.tls=false --metrics --metrics.endpoint otel.celestia.tools:4318 --gateway --gateway.addr localhost --gateway.port 26659 --p2p.network blockspacerace
Restart=on-failure
RestartSec=3
LimitNOFILE=4096
 
[Install]
WantedBy=multi-user.target
EOF
```
9) Start the node:
```
sudo systemctl daemon-reload
sudo systemctl enable celestia-lightd
sudo systemctl start celestia-lightd && sudo journalctl -u celestia-lightd.service -f
```
Check list wallet
```
cd celestia-node
./cel-key list --node.type light --keyring-backend test --p2p.network blockspacerace
```
Lấy light node ID:
```
NODE_TYPE=light
AUTH_TOKEN=$(celestia $NODE_TYPE auth admin --p2p.network blockspacerace)
curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
 ```   
Kết quả trả về chuỗi json kiểu:
```
{"jsonrpc":"2.0","result":{"ID":"12D3KooWSpmuDUXYz1oKj4pj3zVU2UtfmCn8AqyeW4xfXBqJqGLD","Addrs":["/ip4/65.108.156.195/udp/2121/quic-v1","/ip6/2a01:4f9:c011:88a6::1/udp/2121/quic-v1","/ip6/::1/udp/2121/quic-v1","/ip4/65.108.156.195/tcp/2121","/ip6/2a01:4f9:c011:88a6::1/tcp/2121","/ip6/::1/tcp/2121"]},"id":0}
```
ID node của bạn là chuỗi 12D3KooWSpmuDUXYz1oKj4pj3zVU2UtfmCn8AqyeW4xfXBqJqGLD
Check the logs:
```
sudo journalctl -u celestia-lightd.service -f
```

