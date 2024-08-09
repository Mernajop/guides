## **Install Dependencies**

Update system package and install build tools

```bash
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential fail2ban ufw
sudo apt -qy upgrade
```

### **Configure Moniker**

Replace <your-moniker-name> with your own validator name

```bash
MONIKER="<your-moniker-name>"
```

## **Install Go**

install go version 1.20.10

```bash
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.20.10.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```

## **Build Binaries**

Cloning project repository & Compile binaries

```bash
cd $HOME
rm -rf lava
git clone https://github.com/lavanet/lava.git
cd lava
git checkout v2.0.0
```

Export binaries name and build

```bash
export LAVA_BINARY=lavadmake build
```

Prepare binaries for cosmovisor

```bash
mkdir -p $HOME/.lava/cosmovisor/genesis/binmv build/lavad $HOME/.lava/cosmovisor/genesis/bin/rm -rf build
```

Create symlinks

```bash
sudo ln -s $HOME/.lava/cosmovisor/genesis $HOME/.lava/cosmovisor/current -fsudo ln -s $HOME/.lava/cosmovisor/current/bin/lavad /usr/local/bin/lavad -f
```

## **Cosmovisor Setup**

Install cosmovisor

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

## **Create Service**

Create a systemd service

```bash
sudo tee /etc/systemd/system/lava.service > /dev/null << EOF
[Unit]
Description=lava node service
After=network-online.target
 
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.lava"
Environment="DAEMON_NAME=lavad"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.lava/cosmovisor/current/bin"
 
[Install]
WantedBy=multi-user.target
EOF
```

## **Enable Service**

Enable lava systemd service

```bash
sudo systemctl daemon-reloadsudo systemctl enable lava
```

## **Initialize Node**

Setting node configuration

```bash
lavad config chain-id lava-testnet-2
lavad config keyring-backend test
lavad config node tcp://localhost:20457
```

Initialize node

```bash
lavad init $MONIKER --chain-id lava-testnet-2
```

## **Download Genesis & Addrbook**

Download genesis & addrbook file

```bash
curl -Ls https://snap.trekorenthusiast.pics/lava-testnet/genesis.json > $HOME/.lava/config/genesis.json
curl -Ls https://snap.trekorenthusiast.pics/lava-testnet/addrbook.json > $HOME/.lava/config/addrbook.json
```

## **Configure Seeds**

Setting up a seed peers

```bash
sed -i -e "s|^seeds *=.*|seeds = \"d1d43cc7c7aef715957289fd96a114ecaa7ba756@testnet-seeds.trekorenthusiast.pics:20410\"|" $HOME/.lava/config/config.toml
```

## **Configure Gas Prices**

Setting up a gas prices

```bash
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ulava\"|" $HOME/.lava/config/app.toml
```

## **Pruning Setting**

Configure pruning setting

```bash
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.lava/config/app.toml
```

## **Download Snapshots**

Download latest chain snapshot

```bash
curl -L https://snap.trekorenthusiast.pics/lava-testnet/lava-latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.lava
[[ -f $HOME/.lava/data/upgrade-info.json ]] && cp $HOME/.lava/data/upgrade-info.json $HOME/.lava/cosmovisor/genesis/upgrade-info.json
```

## **Start Service**

```bash
sudo systemctl start lava
```
