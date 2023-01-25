# Nolus State Sync

## Info
#### Public RPC endpoint: http://135.181.178.53:43657/
#### Public API: http://135.181.178.53:43317/

## Guide to sync your node using State Sync:

### Copy the entire command
```
sudo systemctl stop nolusd
SNAP_RPC="http://135.181.178.53:43657"; \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash); \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.nolus/config/config.toml

peers="eb7bb7399426f30de769b80a30f804fc1c11df48@135.181.178.53:43656" \
&& sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.nolus/config/config.toml 

nolusd tendermint unsafe-reset-all --home ~/.nolus --keep-addr-book && sudo systemctl restart nolusd && \
journalctl -u nolusd -f --output cat
```

### Turn off State Sync Mode after synchronization
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" $HOME/.nolus/config/config.toml
```
