
# 0G

## Summary

1. Updating OS
2. Installing Dependencies
3. Installing GO
4. Setting up environment variables
5. Initializing and configuring the node
    1. Creating System Link
    2. Downloading Cosmovisor
    3. Node Settings
    4. Initializing the Node
    5. Downloading genesis and addressbook
    6. Setting seeds and peers
    7. Setting custom ports
    8. Configuring pruning settings
    9. Setting gas and other configurations
    10. Creating service file
6. Resetting and downloading latest snap
7. Enabling and starting the service
8. Checking logs
9. Wallet
   1. Creating a New Wallet
   2. Importing existing wallet
   3. Learning your evm adress
   4. Exporting private key
   5. Getting faucet
   6. Network details
10. Creating Validator    
11. Self-Delegating
12. Deleting the node     


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
NODE="0G"
export "NODE=$NODE" >> $HOME/.bash_profile
echo "export OG_NODENAME=$OG_NODENAME"  >> $HOME/.bash_profile
echo "export OG_WALLET=$OG_WALLET" >> $HOME/.bash_profile
echo "export OG_PORT=16" >> $HOME/.bash_profile
echo "export OG_CHAIN_ID=zgtendermint_16600-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Initializing and configuring the node
```bash
git clone -b v0.1.0 https://github.com/0glabs/0g-chain.git
./0g-chain/networks/testnet/install.sh
source .profile
```
```bash
mkdir -p $HOME/.0gchain/cosmovisor/genesis/bin
mv /root/go/bin/0gchaind $HOME/.0gchain/cosmovisor/genesis/bin/
```

### Creating System Link
```bash
sudo ln -s $HOME/.0gchain/cosmovisor/genesis $HOME/.0gchain/cosmovisor/current -f
sudo ln -s $HOME/.0gchain/cosmovisor/current/bin/0gchaind /usr/local/bin/0gchaind -f
```

### Downloading Cosmovisor
```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```
### Node Settings
```bash
0gchaind config chain-id zgtendermint_16600-1
0gchaind config keyring-backend os
0gchaind config node tcp://localhost:16657
```
### Initializing the Node 
```bash
0gchaind init NODE-NAME --chain-id zgtendermint_16600-1
```

### Downloading genesis and addressbook
```bash
rm ~/.0gchain/config/genesis.json
curl -Ls https://github.com/0glabs/0g-chain/releases/download/v0.1.0/genesis.json > $HOME/.0gchain/config/genesis.json
curl -Ls https://raw.githubusercontent.com/kgur88/0G/main/addrbook.json > $HOME/.0gchain/config/addrbook.json
```


### Setting seeds and peers
```bash
PEERS="e166795e85e19674583019303dcdb36ab919dbbb@185.185.82.83:16656,ae1c39dcf8d8a7c956a0333ca3d9176d1df87f64@62.169.23.106:26656,8878193c9c4ed3e03b0041e7a48eb62f5b1c2531@84.247.176.42:16656,b012a5683ca5a10b8bf64667bbedb594880b07d4@95.111.248.207:26656,29faf995bdd64fccc0766ccd173797537e9587d7@31.220.100.221:26656,f0c30cd6024ffea387c86f63139704e28d15bcd8@91.108.226.83:26656,674c780cbc8064f19cad18e3a1d1294c0766eb90@37.27.71.199:20656,aacda82a5b5288ca71640e9d667e666e9ad06ca0@161.97.130.111:26656,6d9da8e218dbeecfb76a6473a1eee073bb62fbdc@38.242.249.0:26656,5ac1856bf0abcb58a1441d91c14741c488f20d9c@88.99.61.53:45656,66a25d8c4b6de583762d02def6e4d0f913f66ffa@15.235.144.20:47656" && \
SEEDS="c4d619f6088cb0b24b4ab43a0510bf9251ab5d7f@54.241.167.190:26656,44d11d4ba92a01b520923f51632d2450984d5886@54.176.175.48:26656,f2693dd86766b5bf8fd6ab87e2e970d564d20aff@54.193.250.204:26656,f878d40c538c8c23653a5b70f615f8dccec6fb9f@54.215.187.94:26656" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.0gchain/config/config.toml
```
### Setting custom ports
```bash
echo "export G_PORT="16"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### Updating ports in app.toml
```bash
sed -i.bak -e "s%:1317%:${G_PORT}317%g;
s%:8080%:${G_PORT}080%g;
s%:9090%:${G_PORT}090%g;
s%:9091%:${G_PORT}091%g;
s%:8545%:${G_PORT}545%g;
s%:8546%:${G_PORT}546%g;
s%:6065%:${G_PORT}065%g" $HOME/.0gchain/config/app.tomll
```

#### Updating ports in config.toml
```bash
sed -i.bak -e "s%:26658%:${G_PORT}658%g;
s%:26657%:${G_PORT}657%g;
s%:6060%:${G_PORT}060%g;
s%:26656%:${G_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${G_PORT}656\"%;
s%:26660%:${G_PORT}660%g" $HOME/.0gchain/config/config.toml
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
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.0gchain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.0gchain/config/config.toml
```


### Creating service file
```bash
sudo tee /etc/systemd/system/0gchaind.service > /dev/null << EOF
[Unit]
Description=0gchaind node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.0gchain"
Environment="DAEMON_NAME=0gchaind"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.0gchain/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```


## Resetting and downloading latest snap
```bash
# Will be added
```

## Enabling and starting the service
### You need to inspect logs visually after the  to make sure that it is working
```bash
sudo systemctl daemon-reload
sudo systemctl enable wardend
sudo systemctl start 0gchaind
```

## Checking logs
```bash
sudo systemctl restart wardend && sudo journalctl -u wardend -f
```

## Wallet 

### Creating a New Wallet
```bash
0gchaind keys add wallet --eth
```

### Importing existing wallet
```bash
0gchaind keys add wallet --eth --recover
```

### Learning your evm adress
```bash
echo "0x$(0gchaind debug addr $(0gchaind keys show wallet -a) | grep hex | awk '{print $3}')"
```
### Exporting private key
```bash
0gchaind keys unsafe-export-eth-key wallet
```

### Getting faucet
Request faucet with your evm adress on website below
https://faucet.0g.ai/

### Network Details 
| Network name	 |0g Newton Testnet|
|----------------|-----------------|
| Rpc Url	|https://rpc-testnet.0g.ai/|
| Chain ID		| 9000 |
| Currency Symbol| A0GI | 

## Creating Validator
```bash
0gchaind tx staking create-validator \
  --amount=1000000ua0gi \
  --pubkey=$(0gchaind tendermint show-validator) \
  --moniker=yourmoniker \
  --chain-id=zgtendermint_16600-1 \
  --commission-rate=0.05 \
  --commission-max-rate=0.10 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation=1 \
  --from=wallet \
  --identity="" \
  --website="" \
  --details="" \
  --node=http://localhost:16657 \
  -y
  ```

## Self-Delegating
```bash
0gchaind tx staking delegate $(0gchaind keys show wallet --bech val -a)  amount000000ua0gi --from wallet -y
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



