# 0G
## Endpoints

|           API            |           RPC            |              
| :----------------------: | :----------------------: | 
| `https://0g.api.kgnodes.xyz` | `https://0g.rpc.kgnodes.xyz/` | 

**Explorer**: [https://explorer.kgnodes.xyz/og](https://explorer.kgnodes.xyz/og)

# Installation


## Update OS

```bash
sudo apt update && sudo apt upgrade -y
```

## Installing Dependencies

```bash
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

## Installing GO

```bash
cd ~
VER="1.21.3" # Make sure that this version does not broke any other apps you run!
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

## Setting up environment variables

```bash
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="yourmoniker"" >> $HOME/.bash_profile
echo "export OG_CHAIN_ID="zgtendermint_16600-2"" >> $HOME/.bash_profile
echo "export OG_PORT="42"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Initializing and configuring the node
### Download Binary
```bash
cd $HOME
rm -rf 0g-chain
git clone -b v0.2.3 https://github.com/0glabs/0g-chain.git
cd 0g-chain
make install
```

### Node Settings

```bash
0gchaind config node tcp://localhost:${OG_PORT}657
0gchaind config keyring-backend file
0gchaind config chain-id zgtendermint_16600-2
```

### Initializing the Node

```bash
0gchaind init "yourmoniker" --chain-id zgtendermint_16600-2
```

### Downloading genesis and addressbook

```bash
wget -O $HOME/.0gchain/config/genesis.json https://snapshot.kgnodes.xyz/snapshot/0gchain/genesis.json
wget -O $HOME/.0gchain/config/addrbook.json  https://snapshot.kgnodes.xyz/snapshot/0gchain/addrbook.json
```

### Setting seeds and peers

```bash
SEEDS="8f21742ea5487da6e0697ba7d7b36961d3599567@og-testnet-seed.itrocket.net:47656"
PEERS="7cf4c84a6e44e919a74f26c04c3bf231fd2d4d03@136.243.93.159:34656,b9d070fd993fd06d7687ca869126dc8410f2bcbd@194.163.158.212:26646,e23db79fe3a7da70288be8fbd3d5f2e2729a6b43@176.57.189.28:26646,bbc62ae5338d7bb55a9b2b4cd9788c8c68c95fd6@178.18.240.50:26646,d17b27dbf66ed4c4f48227371ab1be6399fa6cbe@75.119.158.44:26646,d08764ae3f8c05297d905cffbf18a0d8ff93c169@37.27.127.220:16656,95994c0fbc5821fd835b207a570c85bb654ea803@194.163.183.37:26646,8fa3dccf745f8f66409065405dd502eab78b4a7c@88.198.52.50:34656,5866445694b6dbaf73ea56de2a0c14734b614df9@157.90.209.40:34656,f751dd07516a28076f6f802c16cc12a62e0380b8@168.119.15.124:34656,fe72d1209a7f27eedfabb39555cbedc31c338724@157.90.213.57:34656,8336a508a250320739d1295fca7be2a895ec42c8@94.130.64.31:34656,2037186fdcb85e1c33e89bc30ca343ab07159e37@149.50.101.203:46656,ff5ebf4b036164aa41df114c9713f0f83ffa4370@148.251.46.18:34656,b6a05c26f8d2d18e0e77b2e991845eb43115714c@144.76.30.120:34656,21bf421c20930fd7195b41d1399884c6e693aff9@157.90.213.97:34656"

sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.0gchain/config/config.toml
```

#### Updating ports in app.toml

```bash
sed -i.bak -e "s%:1317%:${OG_PORT}317%g;
s%:8080%:${OG_PORT}080%g;
s%:9090%:${OG_PORT}090%g;
s%:9091%:${OG_PORT}091%g;
s%:8545%:${OG_PORT}545%g;
s%:8546%:${OG_PORT}546%g;
s%:6065%:${OG_PORT}065%g" $HOME/.0gchain/config/app.toml
```

#### Updating ports in config.toml

```bash
sed -i.bak -e "s%:26658%:${OG_PORT}658%g;
s%:26657%:${OG_PORT}657%g;
s%:6060%:${OG_PORT}060%g;
s%:26656%:${OG_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${OG_PORT}656\"%;
s%:26660%:${OG_PORT}660%g" $HOME/.0gchain/config/config.toml
```

### Configuring pruning settings

```bash
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.0gchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.0gchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.0gchain/config/app.toml
```

### Setting gas and other configurations

```bash
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0ua0gi"|g' $HOME/.0gchain/config/app.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.0gchain/config/config.toml
```

### Creating service file

```bash
sudo tee /etc/systemd/system/0gchaind.service > /dev/null <<EOF
[Unit]
Description=0G node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.0gchain
ExecStart=$(which 0gchaind) start --home $HOME/.0gchain
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
## Enabling and starting the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable 0gchaind
sudo systemctl start 0gchaind
```

## Downloading latest snap

### Latest Snapshot:
| **Size** | **Heigt** | **Time** |
|----------|-----------|----------|
|   3.4G       |  459594         |  2024-07-30 01:14 UTC       |
```bash
sudo systemctl stop 0gchaind

cp $HOME/.0gchain/data/priv_validator_state.json $HOME/.0gchain/priv_validator_state.json.backup

0gchaind tendermint unsafe-reset-all --home $HOME/.0gchain --keep-addr-book
curl https://snapshot.kgnodes.xyz/snapshot/0gchain/snap_0gchain.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.0gchain


mv $HOME/.0gchain/priv_validator_state.json.backup $HOME/.0gchain/data/priv_validator_state.json

sudo systemctl restart 0gchaind
sudo journalctl -u 0gchaind -f --no-hostname -o cat
```


## Checking logs

```bash
sudo journalctl -u 0gchaind -f --no-hostname -o cat
```

## Wallet

### Creating a New Wallet

```bash
0gchaind keys add <wallet_name> --keyring-backend file --eth
```

### Importing existing wallet

```bash
0gchaind keys add <wallet_name> --recover --keyring-backend file --eth
```

### Learning your evm adress

```bash
echo "0x$(0gchaind debug addr $(0gchaind keys show <wallet_name> -a) | grep hex | awk '{print $3}')"
```

### Exporting private key

```bash
0gchaind keys unsafe-export-eth-key <wallet_name>
```

### Getting faucet

Request faucet with your evm adress on website below
https://faucet.0g.ai/

### Network Details

| Network name    | 0g Newton Testnet-2         |
| --------------- | -------------------------- |
| Rpc Url         | https://rpc-testnet.0g.ai/ |
| Chain ID        | 9000                       |
| Currency Symbol | A0GI                       |

## Creating Validator

```bash
0gchaind tx staking create-validator \
  --amount=1000000ua0gi \
  --pubkey=$(0gchaind tendermint show-validator) \
  --moniker=yourmoniker \
  --chain-id=zgtendermint_16600-2 \
  --commission-rate=0.1 \
  --commission-max-rate=0.2 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation=1 \
  --from=<wallet_name> \
  --identity="" \
  --website="" \
  --details="" \
  --node=http://localhost:your-rpc-port \
  -y
```

## Self-Delegating

```bash
0gchaind tx staking delegate $(0gchaind keys show <wallet_name> --bech val -a)  amount000000ua0gi --from <wallet_name> --node=http://localhost:your-rpc-port -y
```
## Unjail Validator

```bash
0gchaind tx slashing unjail --from <wallet_name> --chain-id zgtendermint_16600-2 --gas=auto --gas-adjustment=1.6 -y 
```
## Deleting the Node

```bash
systemctl stop 0gchaind && \
systemctl disable 0gchaind && \
rm /etc/systemd/system/0gchaind.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .0gchain 0g-chain && \
rm -rf $(which 0gchaind)
sed -i '/OG_/d' ~/.bash_profile
```
