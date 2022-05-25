## Hardware Requirements

 - Memory: 8 GB RAM
 - CPU: Quad-Core
 - Disk: 250 GB SSD Storage
 - Bandwidth: 1 Gbps for Download/100 Mbps for Upload

## Setting Up Your Validator Node

```
sudo apt update && sudo apt upgrade -y

sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential \
git make ncdu -y

ver="1.17.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile

go version
```
- When we write the go version, the following should come out as a result `go version go1.17.2 linux/amd64`

```
cd $HOME
rm -rf celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app/
APP_VERSION=$(curl -s \
  https://api.github.com/repos/celestiaorg/celestia-app/releases/latest \
  | jq -r ".tag_name")
git checkout tags/$APP_VERSION -b $APP_VERSION
make install
```
```
cd $HOME
celestia-appd init nodename --chain-id mamaki


pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="5000"
pruning_interval="10"

sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \
\"$pruning_keep_recent\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \
\"$pruning_keep_every\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \
\"$pruning_interval\"/" $HOME/.celestia-app/config/app.toml
```
### New Wallet

```celestia-appd keys add walletname --keyring-backend=test```

### Devnet Wallet

```celestia-appd keys add walletname --recover --keyring-backend=test```

- Do not forget to save your wallet information (words)!
- To get a test token, you must enter the Faucet section and type as follows. https://discord.gg/JeRdw5veKu

```
$request walletadress
```
### Continue the Installation

```
wget -O $HOME/.celestia-app/config/genesis.json "https://raw.githubusercontent.com/celestiaorg/networks/master/mamaki/genesis.json"

PEERS="e4429e99609c8c009969b0eb73c973bff33712f9@141.94.73.39:43656\
  09263a4168de6a2aaf7fef86669ddfe4e2d004f6@142.132.209.229:26656,\
  13d8abce0ff9565ed223c5e4b9906160816ee8fa@94.62.146.145:36656,\
  72b34325513863152269e781d9866d1ec4d6a93a@65.108.194.40:26676,\
  322542cec82814d8903de2259b1d4d97026bcb75@51.178.133.224:26666,\
  5273f0deefa5f9c2d0a3bbf70840bb44c65d835c@80.190.129.50:49656,\
  7145da826bbf64f06aa4ad296b850fd697a211cc@176.57.189.212:26656,\
  5a4c337189eed845f3ece17f88da0d94c7eb2f9c@209.126.84.147:26656,\
  ec072065bd4c6126a5833c97c8eb2d4382db85be@88.99.249.251:26656,\
  cd1524191300d6354d6a322ab0bca1d7c8ddfd01@95.216.223.149:26656,\
  2fd76fae32f587eceb266dce19053b20fce4e846@207.154.220.138:26656,\
  1d6a3c3d9ffc828b926f95592e15b1b59b5d8175@135.181.56.56:26656,\
  fe2025284ad9517ee6e8b027024cf4ae17e320c9@198.244.164.11:26656,\
  fcff172744c51684aaefc6fd3433eae275a2f31b@159.203.18.242:26656"
  
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers \
    *=.*/persistent_peers = \"$PEERS\"/" $HOME/.celestia-app/config/config.toml
```
    
```
sed -i.bak -e "s/^timeout-commit *=.*/timeout-commit = \"25s\"/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^skip-timeout-commit *=.*/skip-timeout-commit = false/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^mode *=.*/mode = \"validator\"/" $HOME/.celestia-app/config/config.toml
```

### Create Celestia-App Systemd File

```
sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-appd.service
[Unit]
Description=celestia-appd Cosmos daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/celestia-appd start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
```
cat /etc/systemd/system/celestia-appd.service
```

### Node Start

```
sudo systemctl enable celestia-appd
sudo systemctl start celestia-appd
```

### Check If Daemon Has Been Started Correctly

```
sudo systemctl status celestia-appd
```

### Check Daemon Logs in Real Time

```
sudo journalctl -u celestia-appd.service -f
```

### To check If Your Node is in Sync Before Going Forward

```
curl -s localhost:26657/status | jq .result | jq .sync_info
```
> "catching_up": false

### Connect Validator

```
celestia-appd tx staking create-validator \
    --amount=1000000utia \
    --pubkey=$(celestia-appd tendermint show-validator) \
    --moniker=MonikerAdi \
    --chain-id=mamaki \
    --commission-rate=0.1 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.01 \
    --min-self-delegation=1000000 \
    --from=walletname
```
> confirm transaction before signing and broadcasting [y/N]: y


