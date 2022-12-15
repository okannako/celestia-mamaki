- Mocha testnet is an upgrade of Mamaki testnet. Like Mamaki, there is no reward. This page is for those who are running a Mamaki node and want to upgrade to Mocha.
- Before proceeding, I recommend that you back up your files related to the Mamaki node.

## Hardware Requirements

 - Memory: 8 GB RAM
 - CPU: Quad-Core
 - Disk: 250 GB SSD Storage
 - Bandwidth: 1 Gbps for Download/100 Mbps for Upload

## Mamaki Node Stop and Clean Files
```
sudo systemctl stop celestia-appd
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app
rm -rf celestia-app
rm -rf .celestia-app
```

## Mocha Files
```
cd $HOME
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app/
APP_VERSION=v0.11.0
git checkout tags/$APP_VERSION -b $APP_VERSION
make install
```
```
cd $HOME
rm -rf networks
git clone https://github.com/celestiaorg/networks.git
```
- ``celestia-appd version`` when you type aa, it should output ``0.11.0``

## Mocha Settings
```
celestia-appd init "node-name" --chain-id mocha
cp $HOME/networks/mocha/genesis.json $HOME/.celestia-app/config
SEEDS="8084e73b70dbe7fba3602be586de45a516012e6f@144.76.112.238:26656"
PEERS="eaa763cde89fcf5a8fe44274a5ee3ce24bce2c5b@64.227.18.169:26656, 0d0f0e4a149b50a96207523a5408611dae2796b6@198.199.82.109:26656, c2870ce12cfb08c4ff66c9ad7c49533d2bd8d412@178.170.47.171:26656, d5519e378247dfb61dfe90652d1fe3e2b3005a5b@65.109.68.190:20656, f9e950870eccdb40e2386896d7b6a7687a103c99@88.99.219.120:43656, c2870ce12cfb08c4ff66c9ad7c49533d2bd8d412@178.170.47.171:26656, 8bb8e34ac6eb4ddb927bb1cbbd44357683123af1@188.165.221.155:30542, 0d0f0e4a149b50a96207523a5408611dae2796b6@198.199.82.109:26656, eaa763cde89fcf5a8fe44274a5ee3ce24bce2c5b@64.227.18.169:26656, 3584c49855123abdc16b01a47f9e1bea38a9db1b@154.26.155.102:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.celestia-app/config/config.toml
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

## Reset Network
```
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app
```

## Snapshot
```
cd $HOME
rm -rf ~/.celestia-app/data
mkdir -p ~/.celestia-app/data
SNAP_NAME=$(curl -s https://snaps.qubelabs.io/celestia/ | \
    egrep -o ">mocha.*tar" | tr -d ">")
wget -O - https://snaps.qubelabs.io/celestia/${SNAP_NAME} | tar xf - \
    -C ~/.celestia-app/data/
```

### Wallet
```
celestia-appd keys add walletname
```
```
celestia-appd keys add walletname --recover
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

## Create Validator
```
MONIKER="Validator Name"
VALIDATOR_WALLET="Wallet Name"
EVM_ADDRESS=You can add any Ethereum-based address
ORCHESTRATOR_ADDRESS=Validators certainly can use their existing Celestia addresses here but it is recommended to create a new one.


celestia-appd tx staking create-validator \
    --amount=1000000utia \
    --pubkey=$(celestia-appd tendermint show-validator) \
    --moniker=$MONIKER \
    --chain-id=mocha \
    --commission-rate=0.1 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.01 \
    --min-self-delegation=1000000 \
    --from=$VALIDATOR_WALLET \
    --orchestrator-address=$ORCHESTRATOR_ADDRESS \
    --evm-address=$EVM_ADDRESS \
    --fees 500utia \
    --gas=400671
```
