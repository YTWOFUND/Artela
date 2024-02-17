# Artela

# Artela
Artela Node Installation Instructions </br>
### [Official documentation](https://docs.artela.network/develop/validate/Validators)

### System requirements: </br>
CPU: 4 Core </br>
RAM: 8 Gb </br>
SSD: 200 Gb </br>
OS: Ubuntu 20.04 LTS </br>

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>
    
# Manual configuration of the Lava node with Cosmovisor
Required packages installation </br>
```
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl git jq lz4 build-essential
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```

Go installation.
```
cd $HOME
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
cd $HOME
cd && rm -rf artela
git clone https://github.com/artela-network/artela
cd artela
git checkout v0.4.7-rc6
```

### Build binaries
```
make install
```


# Set node configuration
```
artelad config chain-id artela_11822-1
artelad config keyring-backend test
artelad config node tcp://localhost:26657
```

# Initialize the node
```
artelad init "your moniker" --chain-id artela_11822-1
```

# Download genesis and addrbook
```
curl -L https://snapshots-testnet.nodejumper.io/artela-testnet/genesis.json > $HOME/.artelad/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/artela-testnet/addrbook.json > $HOME/.artelad/config/addrbook.json
```

# Add seeds
```
sed -i -e 's|^seeds *=.*|seeds = "211536ab1414b5b9a2a759694902ea619b29c8b1@47.251.14.47:26656,d89e10d917f6f7472125aa4c060c05afa78a9d65@47.251.32.165:26656,bec6934fcddbac139bdecce19f81510cb5e02949@47.254.24.106:26656,32d0e4aec8d8a8e33273337e1821f2fe2309539a@47.88.58.36:26656,1bf5b73f1771ea84f9974b9f0015186f1daa4266@47.251.14.47:26656"|' $HOME/.artelad/config/config.toml
```

# Set minimum gas price
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "20000000000uart"|' $HOME/.artelad/config/app.toml
```

# Set pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.artelad/config/app.toml
```

### Download latest chain snapshot
```
curl "https://snapshots-testnet.nodejumper.io/artela-testnet/artela-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.artelad"
```

### Create a service
```
sudo tee /etc/systemd/system/artelad.service > /dev/null << EOF
[Unit]
Description=Artela node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which artelad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable artelad.service
```

### Start service and check the logs
```
sudo systemctl start artelad.service && sudo journalctl -u artelad.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
artelad keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
artelad keys add wallet --recover
```

### We receive tokens from the tap in the [discord](https://discord.com/invite/artela)
```
then go to the #testnet-faucet branch and write $request your artela address(0x)
```

### Create the Validator

Before creating a validator, enter the command and check that you have false. This means that the Node has synchronized and you can create a validator:
```
artelad status 2>&1 | jq .SyncInfo.catching_up
```

### Check the balance before creating for the presence of tokens
```
artelad q bank balances $(artelad keys show wallet -a)
```

Please enter your details below in quotation marks where required

```
artelad tx staking create-validator \
--amount=1000000uart \
--pubkey=$(artelad tendermint show-validator) \
--moniker="Your Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I love YTWO" \
--chain-id=artela_11822-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.1uart \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
There have been no updates at the moment, as soon as they come out, we will immediately add them to this section.

Current network:artela_11822-1
Current version:v0.4.7-rc6
```

### Useful commands

Check balance
```
artelad q bank balances $(artelad keys show wallet -a)
```

CHECK SERVICE LOGS
```
sudo journalctl -u artelad -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart artelad
```

GET VALIDATOR INFO
```
artelad status 2>&1 | jq .ValidatorInfo
```

DELEGATE TOKENS TO YOURSELF
```
artelad tx staking delegate $(artelad keys show wallet --bech val -a) 1000000uart --from wallet --chain-id artela_11822-1 --gas-prices 0.1uart --gas-adjustment 1.5 --gas auto -y
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop artelad && sudo systemctl disable artelad && sudo rm /etc/systemd/system/artelad.service && sudo systemctl daemon-reload && rm -rf $HOME/.artelad && rm -rf artela && sudo rm -rf $(which artelad) 
```

