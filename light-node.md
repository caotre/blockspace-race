# Phần cứng
Cấu hình phần cứng yêu cầu tối thiểu:
>Memory: 2 GB RAM. CPU: Single Core. Disk: 5 GB SSD Storage. Bandwidth: 56 Kbps for Download/56 Kbps for Upload

  Link đăng ký VPS Hetzner ủng hộ em https://hetzner.cloud/?ref=fS54U43gi4aS 

# Cài đặt light node
Cài các thành phần cần thiết
```
sudo apt update && sudo apt upgrade -y && sudo apt-get install -y build-essential curl wget jq
```
Cài Go:
```
cd $HOME
wget https://golang.org/dl/go1.19.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.5.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source ~/.bash_profile
```
Kiểm tra phiên bản Go: 
```
go version
```

Tải binary Celestia node:
```
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node/ 
git checkout tags/v0.8.1
make build 
make install 
make cel-key
```
Kiểm tra phiên bản Celestia:
```
celestia version
```
Kết quả trả về
```
Semantic version: v0.8.0 
Commit: ef582655342c73384a66314972428b152227e428 
Build Date: Thu Dec 15 10:19:22 PM UTC 2022 
System version: amd64/linux 
Golang version: go1.19.5 
```

Initialize Light node
```
celestia light init --p2p.network blockspacerace
```
Celestia tự tạo một key tên là my_celes_key. Lưu cẩn thận thông tin MNEMONIC lại. Lưu folder keys trong .celestia-light-blockspacerace-0 cẩn thận.
Địa chỉ ví Celestia của bạn là: celestia1hry8zs6260uwhsxxxxxxxx
```
NAME: my_celes_key
ADDRESS: celestia1hry8zs6260uwhsxxxxxxxx
MNEMONIC (save this somewhere safe!!!): 
xxx xxx xxx
```
Kiểm tra list keys
```
cd celestia-node
./cel-key list --node.type light --p2p.network blockspacerace
```
Kết quả trả về
```
using directory:  /root/.celestia-light-blockspacerace-0/keys
- address: celestia1hry8zs6260uwhsxxxxxxxx
  name: my_celes_key
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"A4w4fdojrqqyIoGpEgIQg2xDxxxx"}'
  type: local
```
Vào Discord kênh **#faucet** lấy token faucet theo format:
```
$request celestia1hry8zs6260uwhsxxxxxxxx...
```  
Tạo service file:
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
Tham số --metrics --metrics.endpoint otel.celestia.tools:4318 đã được thêm sẵn theo yêu cầu của task "Restart Your Node With Metrics Flags for Tracking Uptime".
  Start the node:
```
sudo systemctl daemon-reload
sudo systemctl enable celestia-lightd
sudo systemctl start celestia-lightd && sudo journalctl -u celestia-lightd.service -f
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
{"jsonrpc":"2.0","result":{"ID":"12D3KooWSpmuDUXYz1oKj4pj3zVU2UtfmCn8Aqxxxx","Addrs":["/ip4/65.108.156.195/udp/2121/quic-v1","/ip6/2a01:4f9:c011:88a6::1/udp/2121/quic-v1","/ip6/::1/udp/2121/quic-v1","/ip4/65.108.156.195/tcp/2121","/ip6/2a01:4f9:c011:88a6::1/tcp/2121","/ip6/::1/tcp/2121"]},"id":0}
```
ID node của bạn là chuỗi 12D3KooWSpmuDUXYz1oKj4pj3zVU2UtfmCn8Aqxxxx.
Vào https://tiascan.com/light-nodes dán ID node của bạn vào để kiểm tra trạng thái node

# Others
Check the logs:
```
sudo journalctl -u celestia-lightd.service -f
```
Kết quả logs
```
Mar 31 04:36:11 celestia-light-node celestia[21632]: 2023-03-31T04:36:11.325Z        INFO        header/store        store/store.go:353        new head        {"height": 133077, "hash": "E1BE949AE742384EFCD8DBDA2CEF164707741D6D3955ACF9694A192022783CB7"}
Mar 31 04:36:11 celestia-light-node celestia[21632]: 2023-03-31T04:36:11.325Z        INFO        das        das/subscriber.go:35        new header received via subscription        {"height": 133077}
Mar 31 04:36:11 celestia-light-node celestia[21632]: 2023-03-31T04:36:11.325Z        INFO        das        das/worker.go:80        finished sampling headers        {"from": 133077, "to": 133077, "errors": 0, "finished (s)": 0.000040406}
Mar 31 04:36:14 celestia-light-node celestia[21632]: 2023-03-31T04:36:14.782Z        INFO        canonical-log        swarm/swarm_dial.go:500        CANONICAL_PEER_STATUS: peer=12D3KooWQgxUFfGUpyU7TKcUBx1qM2U6uAquA9oue2y7s3GaeWHR addr=/ip4/38.242.238.84/udp/2121/quic-v1 sample_rate=100 connection_status="established" dir="outbound"
Mar 31 04:36:22 celestia-light-node celestia[21632]: 2023-03-31T04:36:22.033Z        INFO        header/store        store/store.go:353        new head        {"height": 133078, "hash": "609507791010EE48DAB643722A3342DA4266F429FF78232FC289296E03175E77"}
Mar 31 04:36:22 celestia-light-node celestia[21632]: 2023-03-31T04:36:22.033Z        INFO        das        das/subscriber.go:35        new header received via subscription        {"height": 133078}
Mar 31 04:36:22 celestia-light-node celestia[21632]: 2023-03-31T04:36:22.094Z        INFO        das        das/worker.go:80        finished sampling headers        {"from": 133078, "to": 133078, "errors": 0, "finished (s)": 0.060674562}
```
Tạo thêm một key cho node (thay your-key thành tên key bạn muốn)
```
./cel-key add your-key --keyring-backend test --node.type light --p2p.network blockspacerace
```
Khôi phục một key đã tạo trước bằng MNEMONIC (thay your-key thành tên key bạn muốn)
```
./cel-key add your-key --keyring-backend test --node.type light --p2p.network blockspacerace --recover
```
Kiểm tra thông tin node
```
cel_nodeid=12D3KooWSpmuDUXYz1oKj4pj3zVU2UtfmCn8AqyeW4xxxxxxx
curl -s https://leaderboard.celestia.tools/api/v1/nodes/$cel_nodeid | jq
```
```
  "node_id": "12D3KooWSpmuDUXYz1oKj4pj3zVU2UtfmCn8AqyeW4xxxxxxx",
  "node_type": 3,
  "uptime": 99.994415,
  "last_pfb_timestamp": "1970-01-01T00:00:00Z",
  "pfb_count": 0,
  "head": 143317,
  "network_height": 143325,
  "das_latest_sampled_timestamp": "2023-04-01T11:12:21Z",
  "das_network_head": 143317,
  "das_sampled_chain_head": 143317,
  "das_sampled_headers_counter": 102,
  "das_total_sampled_headers": 143317,
  "total_synced_headers": 143316,
  "start_time": "2023-03-31T02:31:18Z",
  "last_restart_time": "2023-04-01T06:37:24Z",
  "node_runtime_counter_in_seconds": 16499,
  "last_accumulative_node_runtime_counter_in_seconds": 102312,
  "node_type_str": "Celestia-Light"
```
# Upgrade
## Upgrade v0.8.1 
Dừng serviced celestia-lightd
```
sudo systemctl stop celestia-lightd
```
Cập nhật bản mới
```
cd celestia-node/ 
git fetch
git checkout tags/v0.8.1 
make build 
make install
```
Xoá các folder blocks, index, data, transients chứa data của bản cũ
```
cd $HOME
cd .celestia-light-blockspacerace-0
sudo rm -rf blocks index data transients
```
init lại light node, keyring ở bản trước đã được tạo nên lần init này celestia-node sẽ không tạo lại keyring
```
celestia light init --p2p.network blockspacerace
```
```
2023-04-01T06:35:33.082Z        INFO    node    nodebuilder/init.go:29  Initializing Light Node Store over '/root/.celestia-light-blockspacerace-0'
2023-04-01T06:35:33.082Z        INFO    node    nodebuilder/init.go:61  Saved config    {"path": "/root/.celestia-light-blockspacerace-0/config.toml"}
2023-04-01T06:35:33.082Z        INFO    node    nodebuilder/init.go:63  Accessing keyring...
2023-04-01T06:35:33.088Z        WARN    node    nodebuilder/init.go:135 Detected plaintext keyring backend. For elevated security properties, consider using the `file` keyring backend
```
Khởi động lại serviced
```
sudo systemctl restart celestia-lightd
sudo journalctl -u celestia-lightd -f
```
```
Apr 01 06:43:46 celestia-light-node celestia[43876]: 2023-04-01T06:43:46.348Z        INFO        header/store        store/store.go:353        new head        {"height": 105473, "hash": "B97463724AF1E6237535142ABB840870C5AD177B34EC120B3BADF3C4919FE42E"}
Apr 01 06:43:46 celestia-light-node celestia[43876]: 2023-04-01T06:43:46.350Z        INFO        das        das/worker.go:80        finished sampling headers        {"from": 105102, "to": 105201, "errors": 0, "finished (s)": 6.936235622}
Apr 01 06:43:46 celestia-light-node celestia[43876]: 2023-04-01T06:43:46.350Z        INFO        das        das/worker.go:80        finished sampling headers        {"from": 104902, "to": 105001, "errors": 0, "finished (s)": 9.652704919}
Apr 01 06:43:46 celestia-light-node celestia[43876]: 2023-04-01T06:43:46.350Z        INFO        das        das/worker.go:80        finished sampling headers        {"from": 105202, "to": 105301, "errors": 0, "finished (s)": 6.936106781}
Apr 01 06:43:46 celestia-light-node celestia[43876]: 2023-04-01T06:43:46.350Z        INFO        das        das/worker.go:80        finished sampling headers        {"from": 105302, "to": 105401, "errors": 0, "finished (s)": 6.936054723}
Apr 01 06:43:46 celestia-light-node celestia[43876]: 2023-04-01T06:43:46.351Z        INFO        das        das/worker.go:80        finished sampling headers        {"from": 105002, "to": 105101, "errors": 0, "finished (s)": 6.937365918}
```
Khi xóa các thư mục "blocks index data transients" ở trên, các blocks của chain phải đồng bộ  lại, do đó bạn sẽ thấy thời gian hoạt động uptime score bằng 0, cho đến khi dữ liệu được đồng bộ lại, khi đó thời gian hoạt động của node của bạn sẽ giống như trước khi nâng cấp. Điều này là bình thường và không có gì phải lo lắng.
