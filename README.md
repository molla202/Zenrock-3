




<h1 align="center"> Zenrock


![image](https://github.com/user-attachments/assets/6874e9ae-034d-4d9b-b93b-aab60756234e)


</h1>


 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>




## 💻 Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	8|
| RAM	| 16+ GB |
| Storage	| 400 GB SSD |

### Public RPC

SOON....

### 🚧Gerekli kurulumlar
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip gcc clang cmake build-essential -y
```

### 🚧 Go kurulumu
```
cd $HOME
VER="1.22.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### 🚧Dosyaları çekelim
```
echo "export ZENROCK_CHAIN_ID="gardia-3"" >> $HOME/.bash_profile
echo "export ZENROCK_PORT="51"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
cd $HOME
wget -O zenrockd.zip https://github.com/Zenrock-Foundation/zrchain/releases/download/v5.5.0/zenrockd.zip
unzip zenrockd.zip
rm zenrockd.zip
chmod +x $HOME/zenrockd
```
```
mkdir -p $HOME/.zrchain/cosmovisor/genesis/bin
cp -r $HOME/zenrockd $HOME/.zrchain/cosmovisor/genesis/bin/zenrockd
```
```
mkdir -p $HOME/.zrchain/cosmovisor/upgrades/v5beta2/bin
sudo mv $HOME/zenrockd $HOME/.zrchain/cosmovisor/upgrades/v5beta2/bin/zenrockd
```
### 🚧System link
```
sudo ln -s $HOME/.zrchain/cosmovisor/genesis $HOME/.zrchain/cosmovisor/current -f
sudo ln -s $HOME/.zrchain/cosmovisor/current/bin/zenrockd /usr/local/bin/zenrockd -f
```
### 🚧Cosmovisor indirelim
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```
### 🚧Servis oluşturalım
```
sudo tee /etc/systemd/system/zenrockd.service > /dev/null << EOF
[Unit]
Description=zenrock node service
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/cosmovisor run start --home $HOME/.zrchain
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=/root/.zrchain"
Environment="DAEMON_NAME=zenrockd"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/root/.zrchain/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target

EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable zenrockd.service
```
### 🚧İnit
NOT: node adınızı yazınız.
```
zenrockd init $MONIKER --chain-id $ZENROCK_CHAIN_ID
zenrockd config set client chain-id $ZENROCK_CHAIN_ID
zenrockd config set client node tcp://localhost:${ZENROCK_PORT}657
```
### 🚧Genesis addrbook 
```
wget -O $HOME/.zrchain/config/genesis.json https://raw.githubusercontent.com/molla202/Zenrock-3/refs/heads/main/genesis.json
wget -O $HOME/.zrchain/config/addrbook.json https://raw.githubusercontent.com/molla202/Zenrock-3/refs/heads/main/addrbook.json
```

### 🚧Gas pruning ayarı
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.zrchain/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.zrchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.zrchain/config/app.toml

sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0urock"|g' $HOME/.zrchain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.zrchain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.zrchain/config/config.toml
```

### 🚧Port Ayarları

```
sed -i.bak -e "s%:1317%:${ZENROCK_PORT}317%g;
s%:8080%:${ZENROCK_PORT}080%g;
s%:9090%:${ZENROCK_PORT}090%g;
s%:9091%:${ZENROCK_PORT}091%g;
s%:8545%:${ZENROCK_PORT}545%g;
s%:8546%:${ZENROCK_PORT}546%g;
s%:6065%:${ZENROCK_PORT}065%g" $HOME/.zrchain/config/app.toml

```
```
sed -i.bak -e "s%:26658%:${ZENROCK_PORT}658%g;
s%:26657%:${ZENROCK_PORT}657%g;
s%:6060%:${ZENROCK_PORT}060%g;
s%:26656%:${ZENROCK_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ZENROCK_PORT}656\"%;
s%:26660%:${ZENROCK_PORT}660%g" $HOME/.zrchain/config/config.toml
```
### 🚧Seed
```
SEEDS="50ef4dd630025029dde4c8e709878343ba8a27fa@zenrock-testnet-seed.itrocket.net:56656"
PEERS="5458b7a316ab673afc34404e2625f73f0376d9e4@zenrock-testnet-peer.itrocket.net:11656,679e513d8f9734018d6019da66c54e19971ff1c3@65.109.22.211:26656,38294f5212c42105eb6679eca9d8b432238ed2a1@65.109.80.26:26656,1dfbd854bab6ca95be652e8db078ab7a069eae6f@52.30.152.47:26656,6ef43e8d5be8d0499b6c57eb15d3dd6dee809c1e@34.246.15.243:26656,12f0463250bf004107195ff2c885be9b480e70e2@52.30.152.47:26656,a412c87699b151a02d1385cbb68b6df396a81c3f@95.217.196.224:26656,ee7d09ac08dc61548d0e744b23e57436b8c477fc@65.109.93.152:26906,e1ff342fb55293384a5e92d4bd3bed82ecee4a60@65.108.234.158:26356,63014f89cf325d3dc12cc8075c07b5f4ee666d64@52.30.152.47:26656,c2c5db24bb7aeb665cbf04c298ca53578043ceed@23.88.0.170:15671,b77eb04d0894d186281943ff6e576ae17acff7ca@213.239.198.181:19656,a746910fab628e98402ca4ebecd4036cf02a2851@51.83.237.195:29556,a98484ac9cb8235bd6a65cdf7648107e3d14dab4@95.217.74.22:18256,c13c25929626d1410a57142ccc5bbf00589666fc@65.108.44.149:14656,fa8356510c907e62eed3c34e13ff26117e4fae6f@65.109.88.19:10056"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.zrchain/config/config.toml
```
### 🚧Snap
```
zenrockd tendermint unsafe-reset-all --home $HOME/.zrchain
if curl -s --head curl http://37.120.189.81/zenrock_testnet/zenrock_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl http://37.120.189.81/zenrock_testnet/zenrock_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.zrchain
    else
  echo "no snapshot found"
fi
```
### 🚧Başlatalım   
```
sudo systemctl daemon-reload
sudo systemctl restart zenrockd
```
### 🚧Log
```
sudo journalctl -u zenrockd.service -f --no-hostname -o cat
```
### 🚧Cüzdan oluşturma
NOT: cüzdan adınızı yazınız
```
zenrockd keys add cuzdan-adini-yaz
```
- Eski cüzdan import ederkene bele
```
zenrockd keys add wallet --recover
```

### 🚧Validator oluşturma

```
cd $HOME
```
```
echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(zenrockd comet show-validator | grep -Po '\"key\":\s*\"\K[^"]*')\"},
    \"amount\": \"1000000urock\",
    \"moniker\": \"test\",
    \"identity\": \"\",
    \"website\": \"\",
    \"security\": \"\",
    \"details\": \"CR\",
    \"commission-rate\": \"0.1\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\"
}" > validator.json
# Create a validator using the JSON configuration
zenrockd tx validation create-validator validator.json \
    --from $WALLET \
    --chain-id gardia-3 \
	--fees 500000urock
```
### Delete
```
sudo systemctl stop zenrockd
sudo systemctl disable zenrockd
sudo rm -rf /etc/systemd/system/zenrockd.service
sudo rm $(which zenrockd)
sudo rm -rf $HOME/.zrchain
sed -i "/ZENROCK_/d" $HOME/.bash_profile
```


