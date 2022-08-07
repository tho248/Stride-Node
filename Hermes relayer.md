# Thiết lập IBC Relayer Hermes giữa Stride và Juno / GAIA
# RPC Endpoint of GAIA fullnode, Juno fullnode
Nếu bạn định thiết lập nút của riêng mình, hãy làm theo hướng dẫn dưới đây từ kjnode

Hướng dẫn thiết lập nút Juno : https://github.com/kj89/testnet_manuals/tree/main/juno Hướng dẫn thiết lập nút GAIA : https://github.com/kj89/testnet_manuals/blob/main/stride/GAIA/README.md

Nếu không, bạn có thể sử dụng một số điểm cuối RPC công khai của GAIA hoặc Juno fullnode . Ở đây giả sử Stride và hermes được cài đặt trên cùng 1 vps , gaia và juno lấy điểm RPC công khai của người khác

##Yêu cầu Stride node của bạn đã đồng bộ hóa hoàn toàn

# Đặt trình chỉ mục thành kv trên mỗi chuỗi

   sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.stride/config/config.toml
   sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.juno/config/config.toml
   sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.gaia/config/config.toml
   
# Hiển thị điểm cuối RPC của bạn cho công chúng,(Nếu Hermes và các fullnode của bạn là cùng một vps, không cần phải làm điều đó)

   sed -i.bak -e 's|^laddr = \"tcp:\/\/.*:\([0-9].*\)57\"|laddr = \"tcp:\/\/0\.0\.0\.0:\157\"|' $HOME/.stride/config/config.toml
   sed -i.bak -e 's|^laddr = \"tcp:\/\/.*:\([0-9].*\)57\"|laddr = \"tcp:\/\/0\.0\.0\.0:\157\"|' $HOME/.juno/config/config.toml
   sed -i.bak -e 's|^laddr = \"tcp:\/\/.*:\([0-9].*\)57\"|laddr = \"tcp:\/\/0\.0\.0\.0:\157\"|' $HOME/.gaia/config/config.toml
   
##Restart lại tất cả node sau khi thay đổi ở bước 3 & 4

# Cài đặt Hermes

   mkdir -p $HOME/.hermes/bin
   cd $HOME/.hermes/bin
   wget https://github.com/informalsystems/ibc-rs/releases/download/v1.0.0-rc.0/hermes-v1.0.0-rc.0-x86_64-unknown-linux-gnu.tar.gz
   tar -C $HOME/.hermes/bin/ -vxzf hermes-v1.0.0-rc.0-x86_64-unknown-linux-gnu.tar.gz
   echo 'export PATH="$HOME/.hermes/bin:$PATH"' >> $HOME/.bash_profile
   source $HOME/.bash_profile
    hermes version
    
# Tạo Config cho Hermes:

Chú ý đến tham số bên dưới rpc_addr, grpc_addr, websocket_addr --Nếu Hermes và fullnode của bạn trên cùng một vps, định dạng của tham số này sẽ là: rpc_addr = ' http: // localhost: RPC_PORT ' (Rerfe bước 5) --Nếu Hermes và fullnode của bạn ở các vps khác nhau, định dạng của tham số này sẽ là: rpc_addr = 'http: // VPS_IP: RPC_PORT' (Tham khảo bước 5) *Tham số key_name phải giống với tên của ví sẽ được thêm vào Hermes sau này
##Ví dụ về Config.toml cho JUNO & STRIDE (STRIDE và HERMES nằm trên cùng một VPS, JUNO nằm trên một VPS khác)

   sudo tee $HOME/.hermes/config.toml > /dev/null <<EOF
   [global]
   log_level = 'info'

   [mode]
   [mode.clients]
   enabled = true
   refresh = true
   misbehaviour = true

   [mode.connections]
   enabled = false

   [mode.channels]
   enabled = false

   [mode.packets]
   enabled = true
   clear_interval = 100
   clear_on_start = true
   tx_confirmation = false

   [rest]
   enabled = true
   host = '127.0.0.1'
   port = 3000

   [telemetry]
   enabled = true
   host = '127.0.0.1'
   port = 3001

   [[chains]]
   id = 'STRIDE-TESTNET-2'
   rpc_addr = 'http://161.97.149.123:26957'
   grpc_addr = 'http://161.97.149.123:9490'
   websocket_addr = 'ws://161.97.149.123:26957/websocket'
   rpc_timeout = '300s'
   account_prefix = 'stride'
   key_name = 'stride-rly'
   address_type = { derivation = 'cosmos' }
   store_prefix = 'ibc'
   default_gas = 100000
   max_gas = 4000000
   gas_price = { price = 0.001, denom = 'ustrd' }
   gas_multiplier = 1.2
   max_msg_num = 30
   max_tx_size = 2097152
   clock_drift = '30s'
   max_block_time = '30s'
   trusting_period = '36000s'
   trust_threshold = { numerator = '1', denominator = '3' }

   memo_prefix = DISDCORD-ID
   [chains.packet_filter]
   policy = 'allow'
   list = [
   ['transfer', '*']
   ]

   [[chains]]
   id = 'uni-3'
   rpc_addr = 'http://159.223.4.92:22657'
   grpc_addr = 'http://159.223.4.92:22090'
   websocket_addr = 'ws://159.223.4.92:22657/websocket'
   rpc_timeout = '300s'
   account_prefix = 'juno'
   key_name = 'juno-rly'
   store_prefix = 'ibc'
   default_gas = 100000
   max_gas = 4000000
   gas_price = { price = 0.001, denom = 'ujunox' }
   gas_multiplier = 1.2
   max_msg_num = 30
   max_tx_size = 2097152
   clock_drift = '30s'
   max_block_time = '30s'
   trusting_period = '2days'
   trust_threshold = { numerator = '1', denominator = '3' }
   memo_prefix = DISDCORD-ID
   address_type = { derivation = 'cosmos' }
   [chains.packet_filter]
   policy = 'allow'
   list = [
   ['transfer', '*']
   ]
   EOF
   
##Config.toml cho GAIA & STRIDE

      sudo tee $HOME/.hermes/config.toml > /dev/null <<EOF
      [global]
      log_level = 'info'

      [mode]
      [mode.clients]
      enabled = true
      refresh = true
      misbehaviour = true

      [mode.connections]
      enabled = false

      [mode.channels]
      enabled = false

      [mode.packets]
      enabled = true
      clear_interval = 100
      clear_on_start = true
      tx_confirmation = true

      [rest]
      enabled = true
      host = '127.0.0.1'
      port = 3000

      [telemetry]
      enabled = true
      host = '127.0.0.1'
      port = 3001

      [[chains]]
      id = 'STRIDE-TESTNET-2'
      rpc_addr = 'http://localhost:16657'
      grpc_addr = 'http://localhost:16090'
      websocket_addr = 'ws://localhost:16657/websocket'
      rpc_timeout = '300s'
      account_prefix = 'stride'
      key_name = 'stride-rly'
      address_type = { derivation = 'cosmos' }
      store_prefix = 'ibc'
      default_gas = 100000
      max_gas = 4000000
      gas_price = { price = 0.001, denom = 'ustrd' }
      gas_multiplier = 1.2
      max_msg_num = 30
      max_tx_size = 2097152
      clock_drift = '25s'
      max_block_time = '30s'
      trusting_period = '36000s'
      trust_threshold = { numerator = '1', denominator = '3' }
      memo_prefix = DISDCORD-ID
      [chains.packet_filter]
      policy = 'allow'
      list = [
      ['ica*', '*'],
      ['transfer', 'channel-0']
      ]

      [[chains]]
      id = 'GAIA'
      rpc_addr = 'http://154.53.44.183:26957'
      grpc_addr = 'http://154.53.44.183:26957'
      websocket_addr = 'ws://localhost:26957/websocket'
      rpc_timeout = '300s'
      account_prefix = 'cosmos'
      key_name = 'gaia-rly'
      address_type = { derivation = 'cosmos' }
      store_prefix = 'ibc'
      default_gas = 100000
      max_gas = 4000000
      gas_price = { price = 0.001, denom = 'uatom' }
      gas_multiplier = 1.2
      max_msg_num = 30
      max_tx_size = 2097152
      clock_drift = '25s'
      max_block_time = '30s'
      trusting_period = '36000s'
      trust_threshold = { numerator = '1', denominator = '3' }
      memo_prefix = 'DISDCORD-ID'
      [chains.packet_filter]
      policy = 'allow'
      list = [
      ['ica*', '*'],
      ['transfer', 'channel-0']
      ]
      EOF
      
# Tạo ví trên mỗi chuỗi, ví phải có fund. Bạn cũng có thể sử dụng ví hiện tại đang được sử dụng cho fullnode / validator
#Đối với ví mới (sử dụng ví cũ thì bỏ qua phần này )

Tạo một cái mới sau đó áp dụng vòi

          strided keys add stride-rly --output json | jq > /root/.hermes/stride-rly.json
          gaiad keys add gaid-rly --output json | jq > /root/.hermes/gaia-rly.json
          junod keys add juno-rly --output json | jq > /root/.hermes/juno-rly.json
#Đối với ví cũ đã có fund

# Tạo tệp json của ví tương ứng trên mỗi chuỗi với định dạng dưới đây
###Export json file

         strided keys show WALLET_NAME --output json | jq > /root/.hermes/stride-rly.json
         gaiad keys show WALLET_NAME --output json | jq > /root/.hermes/gaia-rly.json
         junod keys show WALLET_NAME --output json | jq > /root/.hermes/juno-rly.json
         
###Add 24 seed phrases of wallet into json file, after that the file will be as below
      cat /root/.hermes/stride-rly.json
      {
      "name":"stride-rly",
      "type":"local",
      "address":"WALLET-ADDRESS",
      "pubkey":"{\"@type\":\"/cosmos.crypto.secp256k1.PubKey\",\"key\":\"PUBLIC-KEY\"}",
      "mnemonic":"24 seed phrases"
       }
# Nhập ví vào hermes

      cd /root/.hermes/
      hermes keys add --chain STRIDE-TESTNET-2 --key-file stride-rly.json
      hermes keys add --chain uni-3 --key-file juno-rly.json
      hermes keys add --chain GAIA --key-file gaia-rly.json
      
# Kiểm tra tình trạng và xác thực cấu hình

      hermes health-check
      hermes config validate
      
# Tạo systemd cho Hermes và khởi động nó.

       sudo tee /etc/systemd/system/hermesd.service > /dev/null <<EOF
       [Unit]
       Description=hermes
       After=network-online.target

       [Service]
       User=$USER
       ExecStart=$(which hermes) start
       Restart=on-failure
       RestartSec=3
       LimitNOFILE=65535

       [Install]
       WantedBy=multi-user.target
       EOF

       sudo systemctl daemon-reload
       sudo systemctl enable hermesd
       sudo systemctl restart hermesd
       
# Check Nhật ký giám sát

       sudo journalctl -fu hermesd -o cat
       
#nếu báo lỗi port thì vào file root/.hermes/config.toml đổi port 3000 ---> 3002 , 3001-->3003 và restart lại

# gửi dữ liệu thô giữa 2 bộ chuyển tiếp thông qua ID kênh đã thiết lập (tùy chọn)

###Example for Juno & Stride

      hermes tx raw ft-transfer --dst-chain STRIDE-TESTNET-2 --src-chain uni-3 --src-port transfer --src-channel channel-154 --amount 1000 --denom ujunox --timeout-height-offset 1000 --number-msgs 1
2022-07-28T06:05:10.932566Z INFO ThreadId(01) using default configuration from '/root/.hermes/config.toml' 2022-07-28T06:05:11.017838Z INFO ThreadId(09) wait_for_block_commits: waiting for commit of tx hashes(s) 3DA3A81F21F0BC44D4294D2B0D245DB8213E31AB5A205A133C4A14AFC52D6ABB id=uni-3 Success: [ SendPacket( SendPacket - h:3-987116, seq:3, path:channel-154/transfer->channel-32/transfer, toh:2-14940, tos:Timestamp(NoTimestamp)), ), ]

        hermes tx raw ft-transfer --dst-chain uni-3 --src-chain STRIDE-TESTNET-2 --src-port transfer --src-channel channel-32 --amount 1000 --denom ustrd --timeout-height-offset 1000 --number-msgs 1
2022-07-28T06:07:22.864250Z INFO ThreadId(01) using default configuration from '/root/.hermes/config.toml' 2022-07-28T06:07:33.214633Z INFO ThreadId(09) wait_for_block_commits: waiting for commit of tx hashes(s) 2A31B32C77DE2AE9564C0F41A28BC914DF08040A0B503319E801B78121E1AFEB id=STRIDE-TESTNET-2 Success: [ SendPacket( SendPacket - h:2-13951, seq:4, path:channel-32/transfer->channel-154/transfer, toh:3-988135, tos:Timestamp(NoTimestamp)), ), ]

###Example for GAIA & Stride

          hermes tx raw ft-transfer --dst-chain GAIA --src-chain STRIDE-TESTNET-2 --src-port transfer --src-channel channel-0 --amount 1000 --denom ustrd --timeout-height-offset 1000 --number-msgs 1
2022-07-28T10:54:56.184868Z INFO ThreadId(01) using default configuration from '/root/.hermes/config.toml' 2022-07-28T10:54:58.342749Z INFO ThreadId(10) wait_for_block_commits: waiting for commit of tx hashes(s) 93CD35E9E449ACD34EA74C31D0E9C691709F610003FC6630D1027179DDDE312C id=STRIDE-TESTNET-2 Success: [ SendPacket( SendPacket - h:2-15333, seq:580, path:channel-0/transfer->channel-0/transfer, toh:0-25850, tos:Timestamp(NoTimestamp)), ), ] hermes tx raw ft-transfer --dst-chain STRIDE-TESTNET-2 --src-chain GAIA --src-port transfer --src-channel channel-0 --amount 1000 --denom uatom --timeout-height-offset 1000 --number-msgs 1

2022-07-28T10:55:45.959731Z INFO ThreadId(01) using default configuration from '/root/.hermes/config.toml' 2022-07-28T10:55:46.607686Z INFO ThreadId(11) wait_for_block_commits: waiting for commit of tx hashes(s) 4BB7B4D671D69F95E69B5D88361C9724BA8FE0B1D2CAABAAC289D43DE2D368EF id=GAIA Success: [ SendPacket( SendPacket - h:0-24861, seq:692, path:channel-0/transfer->channel-0/transfer, toh:2-16337, tos:Timestamp(NoTimestamp)), ), ]

# To explorer check txhash update clien Success