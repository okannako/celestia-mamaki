- It is the installation guide for Mocha-3 from scratch.

## Hardware Requirements

 - Memory: 8 GB RAM
 - CPU: Quad-Core
 - Disk: 250 GB SSD Storage
 - Bandwidth: 1 Gbps for Download/100 Mbps for Upload

## Update
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git make ncdu -y
sudo systemctl restart systemd-journald
```

## Golang Install
```
ver="1.20.3"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```
- When we write the go version, the following should come out as a result `go version go1.20.3 linux/amd64`

## Mocha Files
```
cd $HOME
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app/
APP_VERSION=v1.0.0-rc10
git checkout tags/$APP_VERSION -b $APP_VERSION
make install
```
```
cd $HOME
rm -rf networks
git clone https://github.com/celestiaorg/networks.git
```
- ``celestia-appd version`` when you type aa, it should output ``v1.0.0-rc10``

## Mocha Settings
```
celestia-appd init "node-name" --chain-id mocha-3
cp $HOME/networks/mocha-3/genesis.json $HOME/.celestia-app/config
SEEDS="9aa8a73ea9364aa3cf7806d4dd25b6aed88d8152@celestia-testnet.seed.mzonder.com:11156,258f523c96efde50d5fe0a9faeea8a3e83be22ca@seed.mocha-3.celestia.aviaone.com:20275,3314051954fc072a0678ec0cbac690ad8676ab98@65.108.66.220:26656"
sed -i "s|^seeds *=.*|seeds = \"$SEEDS\"|" $HOME/.celestia-app/config/config.toml
PEERS="ec11f3be74010b78882de2cbd170d7ad4458d8ac@157.245.250.63:26656,ec11f3be74010b78882de2cbd170d7ad4458d8ac@157.245.250.63:26656,5073ad517afaeae51cf939381d7dce09880f47f6@51.158.76.4:26656,9c94e40188137f9b1314c205dd9a25fa62b7688e@198.27.82.6:26656,5ec7477a55b48984ec778bd1bef87d2ac8cf95eb@138.201.60.238:26656,e409695e0fcd1a2b521bae27621dca4686cb1ccc@198.199.70.16:26656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.celestia-app/config/config.toml
sed -i 's/^timeout_commit *=.*/timeout_commit = "11s"/' $HOME/.celestia-app/config/config.toml
```

## Configure Pruning
```
PRUNING="custom"
PRUNING_KEEP_RECENT="100"
PRUNING_INTERVAL="10"

sed -i -e "s/^pruning *=.*/pruning = \"$PRUNING\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \
\"$PRUNING_KEEP_RECENT\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \
\"$PRUNING_INTERVAL\"/" $HOME/.celestia-app/config/app.toml
```

## Create Celestia-App Systemd File
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

## Node Start
```
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app
sudo systemctl enable celestia-appd
sudo systemctl start celestia-appd
```

## Check If Daemon Has Been Started Correctly
```
sudo systemctl status celestia-appd
```
- You can exit the status screen by pressing `ctrl+c` 

## Check Daemon Logs in Real Time

```
sudo journalctl -u celestia-appd.service -f
```

## To check If Your Node is in Sync Before Going Forward
```
curl -s localhost:26657/status | jq .result | jq .sync_info
```
> latest_block_height =  compare its value with the explorer value. When they are identical to each other, synchronization to the system is complete.

## Wallet
```
celestia-appd keys add walletname
```
```
celestia-appd keys add walletname --recover
```
- Do not forget to save your wallet information (words)!
- To get a test token, you must enter the Faucet section and type as follows. https://discord.gg/JeRdw5veKu

## Create Validator
```
MONIKER="Validator Name"
VALIDATOR_WALLET="Wallet Name"
EVM_ADDRESS=You can add any Ethereum-based address

celestia-appd tx staking create-validator \
    --amount=1000000utia \
    --pubkey=$(celestia-appd tendermint show-validator) \
    --moniker=$MONIKER \
    --chain-id=mocha-3 \
    --commission-rate=0.1 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.01 \
    --min-self-delegation=1000000 \
    --from=$VALIDATOR_WALLET \
    --evm-address=$EVM_ADDRESS \
    --fees=990utia \
    --gas=900000
```

## Delegate to a Validator
```
celestia-appd tx staking delegate VALIDATOR_ADDRESS 1000000utia --chain-id mocha-3 --fees=300utia --from walletname
```
