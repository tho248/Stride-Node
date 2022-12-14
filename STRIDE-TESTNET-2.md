# Guide setup STRIDE-TESTNET-2
# STRIDE-1 -> STRIDE-TESTNET-2
       cd
       sudo systemctl stop strided
       strided tendermint unsafe-reset-all --home $HOME/.stride
       rm -rf $HOME/stride $HOME/go/bin/strided $HOME/.stride/config/genesis.json
       git clone https://github.com/Stride-Labs/stride.git
       cd stride
       git checkout 644c7574ee79128970a81cf8b9f23351dcdeec62
       mkdir -p $HOME/go/bin
       go build -mod=readonly -trimpath -o $HOME/go/bin ./...
       wget -O $HOME/.stride/config/addrbook.json "https://raw.githubusercontent.com/StakeTake/guidecosmos/main/stride/STRIDE-TESTNET-2/addrbook.json"
       wget -O $HOME/.stride/config/genesis.json "https://raw.githubusercontent.com/Stride-Labs/testnet/main/poolparty/genesis.json"
       sudo systemctl restart strided && journalctl -u strided -f -o cat
   
# Start with state sync

       sudo systemctl stop strided
   
       strided tendermint unsafe-reset-all --home $HOME/.stride
   
       SEEDS=""; \
       PEERS="48b1310bc81deea3eb44173c5c26873c23565d33@34.135.129.186:26656,0f45eac9af97f4b60d12fcd9e14a114f0c085491@stride-library.poolparty.stridenet.co:26656"; \
       sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.stride/config/config.toml
   
       wget -O $HOME/.stride/config/addrbook.json "https://raw.githubusercontent.com/StakeTake/guidecosmos/main/stride/STRIDE-TESTNET-2/addrbook.json"
   
       SNAP_RPC="http://stride.stake-take.com:26657"
   
       LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
       BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
       TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
   
       echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

       sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
       s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
       s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
       s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
       s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.stride/config/config.toml
   
       sudo systemctl restart strided && journalctl -u strided -f -o cat
   
   #check sync ,if "catching_up??? = false is ok
   
               strided status 2>&1 | jq .SyncInfo
               
# Create your validator
       strided tx staking create-validator \
       --amount=9000000ustrd \
       --pubkey=$(strided tendermint show-validator) \
       --moniker=<YOUR-MONIKER> \
       --chain-id=STRIDE-TESTNET-2 \
       --commission-rate="0.07" \
       --commission-max-rate="0.1" \
       --commission-max-change-rate="0.05" \
       --min-self-delegation="1" \
       --fees=250ustrd \
       --gas=200000 \
       --from=<YOUR-ADDRESS-NAME> \
       --details="YOUR-OPTION" \
       -y
