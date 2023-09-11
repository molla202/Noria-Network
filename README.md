![logo-blue-orange-dot](https://github.com/molla202/Noria-Network/assets/91562185/fc248109-266b-433c-b5a0-3ab329bb0821)



<h1 align="center"> NORIA NETWORK </h1>

 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>
 * [Core Node Site](https://corenode.info/)<br>
 * [AltheaNetwork Website](https://noria.network/)<br>
 * [Blockchain Explorer](https://app.noria.network/noria)<br>
 * [Discord](https://discord.gg/cMVTyavpNN)<br>
 * [AltheaNetwork Twitter](https://twitter.com/NoriaNetwork)<br>


## Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| ✔️CPU |	4 |
| ✔️RAM	| 8 GB |
| ✔️Storage	| 250 GB SSD |
| ✔️UBUNTU | 22 |

Not: discorda girip ticket olusturup validator için cüzdan adresi atıp isteyin.... ÖDÜLSÜZDÜR...

### Update ve kütüphane kurulumu
```
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

```
### Go kurulumu yapalım
```
cd $HOME
! [ -x "$(command -v go)" ] && {
VER="1.19.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
}
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```
```
# Dosyaları çekelim
cd $HOME
rm -rf noria
git clone https://github.com/noria-net/noria.git
cd noria
git checkout v1.3.0

# Kuralım
make build
```
```
# Binaries Cosmovisor ayarları
mkdir -p $HOME/.noria/cosmovisor/genesis/bin
mv build/noriad $HOME/.noria/cosmovisor/genesis/bin/
rm -rf build

# Taşıma işlemleri
sudo ln -s $HOME/.noria/cosmovisor/genesis $HOME/.noria/cosmovisor/current -f
sudo ln -s $HOME/.noria/cosmovisor/current/bin/noriad /usr/local/bin/noriad -f
```
```
# Cosmovisor indir
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0

# Servis ayarlaması
sudo tee /etc/systemd/system/noriad.service > /dev/null << EOF
[Unit]
Description=noria node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.noria"
Environment="DAEMON_NAME=noriad"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.noria/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable noriad
```
```
# Configurasyon
noriad config chain-id oasis-3
noriad config keyring-backend test
noriad config node tcp://localhost:16157
```
```
# Moniker adınızı giriniz
noriad init $MONIKER --chain-id oasis-3
```
```
#  Genesis ve addrbook
curl -Ls https://snapshots.kjnodes.com/noria-testnet/genesis.json > $HOME/.noria/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/noria-testnet/addrbook.json > $HOME/.noria/config/addrbook.json

# Seeds
sed -i -e "s|^seeds *=.*|seeds = \"3f472746f46493309650e5a033076689996c8881@noria-testnet.rpc.kjnodes.com:16159\"|" $HOME/.noria/config/config.toml

# Gas ayarı
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0025ucrd\"|" $HOME/.noria/config/app.toml

# Puring
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.noria/config/app.toml
```
```
# Port ayarları
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:16158\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:16157\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:16160\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:16156\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":16166\"%" $HOME/.noria/config/config.toml
sed -i -e "s%^address = \"tcp://localhost:1317\"%address = \"tcp://0.0.0.0:16117\"%; s%^address = \":8080\"%address = \":16180\"%; s%^address = \"localhost:9090\"%address = \"0.0.0.0:16190\"%; s%^address = \"localhost:9091\"%address = \"0.0.0.0:16191\"%; s%:8545%:16145%; s%:8546%:16146%; s%:6065%:16165%" $HOME/.noria/config/app.toml
```
### Snap isterseniz...
```
curl -L https://snapshots.kjnodes.com/noria-testnet/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.noria
[[ -f $HOME/.noria/data/upgrade-info.json ]] && cp $HOME/.noria/data/upgrade-info.json $HOME/.noria/cosmovisor/genesis/upgrade-info.json
```
### Cüzdan oluşturma yada import etme
* Yeni cüzdan
```
noriad keys add wallet
```
* İmport
```
noriad keys add wallet --recover
```
### ve başlatalım
```
sudo systemctl start noriad && sudo journalctl -u noriad -f --no-hostname -o cat
```

### Validor oluşturma
```
noriad tx staking create-validator \
--amount 1000000unoria \
--pubkey $(noriad tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id oasis-3 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.0025ucrd \
-y
```

### Unjail
```
noriad tx slashing unjail --from wallet --chain-id oasis-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.0025ucrd -y
```
### Kendine delege
```
noriad tx staking delegate $(noriad keys show wallet --bech val -a) 1000000unoria --from wallet --chain-id oasis-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.0025ucrd -y
```
