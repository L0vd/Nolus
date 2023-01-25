<p style="font-size:14px" align="right">
<a href="https://t.me/L0vd_staking" target="_blank">Join our telegram <img src="https://raw.githubusercontent.com/L0vd/screenshots/main/Telegram_logo.png" width="30"/></a>
<a href="https://l0vd.com/" target="_blank">Visit our website <img src="https://raw.githubusercontent.com/L0vd/screenshots/main/L0vd.png" width="30"/></a>
</p>



# Table of contents <br />
[Node setup](#node_setup) <br />
[State Sync](#state_sync) <br />
[Starting a validator](#starting_validator) <br />
[Useful commands](#useful_commands)



<a name="node_setup"></a>
# Manual node setup
If you want to setup Nolus fullnode manually follow the steps below

## Update and install dependencies
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc -y
```

## Install GO
```
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.3"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Install node
```
cd $HOME
git clone https://github.com/Nolus-Protocol/nolus-core
cd nolus-core
git checkout v0.1.39
make install
nolusd version #0.1.39
```


## Setting up vars
You should replace values in <> <br />
<YOUR_MONIKER> Here you should put name of your moniker (validator) that will be visible in explorer <br />
<YOUR_WALLET> Here you shoud put the name of your wallet

```
echo "export NOLUS_WALLET="<YOUR_WALLET_NAME>"" >> $HOME/.bash_profile
echo "export NOLUS_NODENAME="<YOUR_MONIKER>"" >> $HOME/.bash_profile
echo "export NOLUS_CHAIN_ID="nolus-rila"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```


## Configure your node
```
nolusd config chain-id $NOLUS_CHAIN_ID
```

## Initialize your node
```
nolusd init $NODENAME --chain-id $CHAIN_ID
```

## Download genesis and addrbook
```
wget -O $HOME/.nolus/config/genesis.json "https://raw.githubusercontent.com/L0vd/Nolus/main/Node_installation_guide/genesis.json"
wget -O $HOME/.nolus/config/addrbook.json "https://raw.githubusercontent.com/L0vd/Nolus/main/Node_installation_guide/addrbook.json"
```

## (OPTIONAL) Set custom ports

### If you want to use non-default ports
```
NOLUS_PORT=<SET_CUSTOM_PORT> #Example: NOLUS_PORT=56 (numbers from 1 to 64)
```
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${NOLUS_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${NOLUS_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${NOLUS_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${NOLUS_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${NOLUS_PORT}660\"%" $HOME/.nolus/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${NOLUS_PORT}317\"%; s%^address = \":8080\"%address = \":${NOLUS_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${NOLUS_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${NOLUS_PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${NOLUS_PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${NOLUS_PORT}546\"%" $HOME/.nolus/config/app.toml
```


## Set seeds and peers
```
SEEDS=""
PEERS="1df6735ac39c8f07ae5db31923a0d38ec6d1372b@45.136.40.6:26656,9726b7ba17ee87006055a9b7a45293bfd7b7f0fc@45.136.40.16:26656,6e84cde074d4af8a9df59d125db3bf8d6722a787@45.136.40.18:26656,eda3e2255f3c88f97673d61d6f37b243de34e9d9@45.136.40.13:26656,4de8c8acccecc8e0bed4a218c2ef235ab68b5cf2@45.136.40.12:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.nolus/config/config.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.nolus/config/app.toml
```

## Set minimum gas price and timeout commit
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0unls\"/" $HOME/.nolus/config/app.toml
```

## Create Service
```
sudo tee /etc/systemd/system/nolusd.service > /dev/null <<EOF
[Unit]
Description=Lava
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nolusd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Reset blockchain info and restart your node
```
sudo systemctl daemon-reload
sudo systemctl enable nolusd
nolusd tendermint unsafe-reset-all --home $HOME/.nolus --keep-addr-book
sudo systemctl restart nolusd && sudo journalctl -u nolusd -f -o cat
```

<a name="state_sync"></a>
## (OPTIONAL) Use State Sync

### [State Sync guide](https://github.com/L0vd/Nolus/tree/main/StateSync)


<a name="starting_validator"></a>
## Starting a validator

### 1. Add a new key
```
nolusd keys add $NOLUS_WALLET
```
#### (OR)

### 1. Recover your key
```
nolusd keys add $NOLUS_WALLET --recover
```

### 2. Request tokens from [faucet](https://discord.com/channels/999302051538411671/1039540296540770385)

### 3. Create validator
```
nolusd tx staking create-validator \
--amount 1000000unls \
--commission-max-change-rate "0.1" \
--commission-max-rate "0.20" \
--commission-rate "0.1" \
--min-self-delegation "1" \
--details "" \
--pubkey=$(nolusd tendermint show-validator) \
--moniker $NOLUS_NODENAME \
--chain-id $NOLUS_CHAIN_ID \
--gas-prices 0.025unls \
--from $NOLUS_WALLET \
--yes
```
<a name="useful_commands"></a>
## Useful commands

### Check status
```
nolusd status | jq
```

### Check logs
```
sudo journalctl -u nolusd -f
```

### Check wallets
```
nolusd keys list
```

### Check balance
```
nolusd q bank balances $NOLUS_WALLET
```

### Send tokens
```
nolusd tx bank send <FROM_WALLET_ADDRESS> <TO_WALLET_ADDRESS> <AMOUNT>unls --fees 1000unls
```

### Delegate tokens to validator
```
nolusd tx staking delegate <MONIKER> <AMOUNT>unls --from $NOLUS_WALLET --chain-id $NOLUS_CHAIN_ID --fees 1000unls
```

### Vote for proposal
#### Yes
```
nolusd tx gov vote <PROPOSAL_NUMBER> yes --from $NOLUS_WALLET --chain-id $NOLUS_CHAIN_ID --fees 1000unls
```
#### No
```
nolusd tx gov vote <PROPOSAL_NUMBER> no --from $NOLUS_WALLET --chain-id $NOLUS_CHAIN_ID --fees 1000unls
```
