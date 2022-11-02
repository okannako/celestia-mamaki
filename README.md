## Hardware Requirements

 - Memory: 8 GB RAM
 - CPU: Quad-Core
 - Disk: 250 GB SSD Storage
 - Bandwidth: 1 Gbps for Download/100 Mbps for Upload

## Setting Up Your Validator Node

```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential \
git make ncdu -y
```
```
ver="1.18.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```
- When we write the go version, the following should come out as a result `go version go1.18.2 linux/amd64`

```
cd $HOME
rm -rf celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app
```
```
git pull
git checkout v0.6.0
make install
```
```
celestia-appd version
```

```
cd $HOME
celestia-appd init nodename --chain-id mamaki
```
```
pruning="custom"
pruning_keep_recent="100"
pruning_interval="10"
```
```
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.celestia-app/config/app.toml
```
```
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \
\"$pruning_keep_recent\"/" $HOME/.celestia-app/config/app.toml
```
```
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \
\"$pruning_interval\"/" $HOME/.celestia-app/config/app.toml
```
### New Wallet

```celestia-appd keys add walletname```

### Devnet Wallet

```celestia-appd keys add walletname --recover```

- Do not forget to save your wallet information (words)!
- To get a test token, you must enter the Faucet section and type as follows. https://discord.gg/JeRdw5veKu

```
$request walletadress
```
### Continue the Installation

```
wget -O $HOME/.celestia-app/config/genesis.json "https://raw.githubusercontent.com/celestiaorg/networks/master/mamaki/genesis.json"
```
```
BOOTSTRAP_PEERS=$(curl -sL https://raw.githubusercontent.com/celestiaorg/networks/master/mamaki/bootstrap-peers.txt | tr -d '\n') && echo $BOOTSTRAP_PEERS
sed -i.bak -e "s/^bootstrap-peers *=.*/bootstrap-peers = \"$BOOTSTRAP_PEERS\"/" $HOME/.celestia-app/config/config.toml
```
```
PEERS="f7b68a491bae4b10dbab09bb3a875781a01274a5@65.108.199.79:20356, 853a9fbb633aed7b6a8c759ba99d1a7674b706a3@38.242.216.151:26656, 96995456b7fe3ab6524fc896dec76d9ba79d420c@212.125.21.178:26656, 268694eaf9446b2052b1161979bf2e09f3e45e81@173.212.254.166:26656, 28aaa8865f3e9bba69f257b08d5c28091b5b3167@176.57.150.79:26656"
sed -i.bak -e "s/^persistent-peers *=.*/persistent-peers = \"$PEERS\"/" $HOME/.celestia-app/config/config.toml
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
```
cd $HOME/.celestia-app
celestia-appd tendermint unsafe-reset-all --home "$HOME/.celestia-app"
```
### Quick-Sync With Snapshot (This episode is not in the video. But there is no difference. You can enter the codes in order)

```
cd $HOME
rm -rf ~/.celestia-app/data
mkdir -p ~/.celestia-app/data
```
```
SNAP_NAME=$(curl -s https://snaps.qubelabs.io/celestia/ | \
    egrep -o ">mamaki.*tar" | tr -d ">")
```
```
wget -O - https://snaps.qubelabs.io/celestia/${SNAP_NAME} | tar xf - \
    -C ~/.celestia-app/data/
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
- You can exit the status screen by pressing `ctrl+c` 

### Check Daemon Logs in Real Time

```
sudo journalctl -u celestia-appd.service -f
```

### To check If Your Node is in Sync Before Going Forward

```
curl -s localhost:26657/status | jq .result | jq .sync_info
```
> latest_block_height =  compare its value with the explorer value. When they are identical to each other, synchronization to the system is complete.

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

### Delegate to a Validator

```
celestia-appd tx staking delegate VALIDATOR_ADDRESS 1000000utia --chain-id mamaki --fees=1utia --from walletname
```

### Unjail

```
celestia-appd tx slashing unjail --from=walletname --chain-id mamaki
```

### See The Number of Peers You Are Connected to

```
curl -sS http://localhost:26657/net_info | jq -r '.result.n_peers'
```

### Node Restart
```
sudo systemctl restart celestia-appd
```

### Find Out The Node Address

```
celestia-appd keys show walletname --bech=val
```

### No Journal Error
```
systemctl restart systemd-journald.service
```

### Delete Node

```
cd
rm -rf celestia-app
rm -rf .celestia-app
```

### Node Setup Video
   *  https://www.youtube.com/watch?v=VH9Ar6phK00&t=10s
