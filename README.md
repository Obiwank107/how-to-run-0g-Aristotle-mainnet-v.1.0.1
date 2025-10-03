# how-to-run-0g-Aristotle-mainnet-v.1.0.1 validator
guide how to run 0g-Aristotle(mainnet) v.1.0.1 validator

# install go, if needed
```
cd $HOME
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

# set vars in the < > as you need
```
echo "export MONIKER="<test>"" >> $HOME/.bash_profile
echo "export OG_PORT="<26>"" >> $HOME/.bash_profile
echo 'export PATH=$PATH:$HOME/galileo-used/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
```

```
cd $HOME
rm -rf aristotle
wget -O aristotle.tar.gz https://github.com/0gfoundation/0gchain-Aristotle/releases/download/1.0.1/aristotle-v1.0.1.tar.gz
tar -xzvf aristotle.tar.gz -C $HOME
rm -rf $HOME/aristotle.tar.gz
mv aristotle-v1.0.1 aristotle
chmod +x $HOME/aristotle/bin/geth
chmod +x $HOME/aristotle/bin/0gchaind
cp $HOME/aristotle/bin/geth $HOME/go/bin/geth
cp $HOME/aristotle/bin/0gchaind $HOME/go/bin/0gchaind
```
```
mv $HOME/aristotle $HOME/aristotle-used
```
```
mkdir -p $HOME/.0gchaind
cp -r $HOME/aristotle-used/0g-home $HOME/.0gchaind
```
```
geth init --datadir $HOME/.0gchaind/0g-home/geth-home $HOME/aristotle-used/geth-genesis.json
```
```
0gchaind init $MONIKER --home $HOME/.0gchaind/tmp
mv $HOME/.0gchaind/tmp/data/priv_validator_state.json $HOME/.0gchaind/0g-home/0gchaind-home/data/
mv $HOME/.0gchaind/tmp/config/node_key.json $HOME/.0gchaind/0g-home/0gchaind-home/config/
mv $HOME/.0gchaind/tmp/config/priv_validator_key.json $HOME/.0gchaind/0g-home/0gchaind-home/config/
rm -rf $HOME/.0gchaind/tmp
```
# Set moniker in config.toml file
```
sed -i -e "s/^moniker *=.*/moniker = \"$MONIKER\"/" $HOME/.0gchaind/0g-home/0gchaind-home/config/config.toml
```
# set custom ports in geth-config.toml file
```
sed -i "s/HTTPPort = .*/HTTPPort = ${OG_PORT}545/" $HOME/aristotle-used/geth-config.toml
sed -i "s/WSPort = .*/WSPort = ${OG_PORT}546/" $HOME/aristotle-used/geth-config.toml
sed -i "s/AuthPort = .*/AuthPort = ${OG_PORT}551/" $HOME/aristotle-used/geth-config.toml
sed -i "s/ListenAddr = .*/ListenAddr = \":${OG_PORT}303\"/" $HOME/aristotle-used/geth-config.toml
sed -i "s/^# *Port = .*/# Port = ${OG_PORT}901/" $HOME/aristotle-used/geth-config.toml
sed -i "s/^# *InfluxDBEndpoint = .*/# InfluxDBEndpoint = \"http:\/\/localhost:${OG_PORT}086\"/" $HOME/aristotle-used/geth-config.toml
```
# set custom ports in config.toml file
```
sed -i.bak -e "s%:26658%:${OG_PORT}658%g;
s%:26657%:${OG_PORT}657%g;
s%:6060%:${OG_PORT}060%g;
s%:26656%:${OG_PORT}656%g;
s%:26660%:${OG_PORT}660%g" $HOME/.0gchaind/0g-home/0gchaind-home/config/config.toml
```
# set custom ports in app.toml file
```
sed -i "s/address = \".*:3500\"/address = \"127\.0\.0\.1:${OG_PORT}500\"/" $HOME/.0gchaind/0g-home/0gchaind-home/config/app.toml
sed -i "s/^rpc-dial-url *=.*/rpc-dial-url = \"http:\/\/localhost:${OG_PORT}551\"/" $HOME/.0gchaind/0g-home/0gchaind-home/config/app.toml
```
# disable indexer
```
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.0gchaind/0g-home/0gchaind-home/config/config.toml
```
# configure pruning
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.0gchaind/0g-home/0gchaind-home/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.0gchaind/0g-home/0gchaind-home/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.0gchaind/0g-home/0gchaind-home/config/app.toml
```
# Create simlink
```
ln -sf $HOME/.0gchaind/0g-home/0gchaind-home/config/client.toml $HOME/.0gchaind/config/client.toml
```
# Gen jwt.hex
```
echo "0x$(openssl rand -hex 32)" > /root/aristotle-used/jwt.hex
chmod 600 /root/aristotle-used/jwt.hex
```
# Create service
```
sudo tee /etc/systemd/system/0ggeth.service > /dev/null <<EOF
[Unit]
Description=0g Geth Node Service
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/aristotle-used
ExecStart=$HOME/go/bin/geth \
    --config $HOME/aristotle-used/geth-config.toml \
    --datadir $HOME/.0gchaind/0g-home/geth-home \
    --networkid 16661 \
    --http.port ${OG_PORT}545 \
    --ws.port ${OG_PORT}546 \
    --authrpc.port ${OG_PORT}551 \
    --bootnodes enode://2bf74c837a98c94ad0fa8f5c58a428237d2040f9269fe622c3dbe4fef68141c28e2097d7af6ebaa041194257543dc112514238361a6498f9a38f70fd56493f96@8.221.140.134:30303 \
    --port ${OG_PORT}303 \
    --nat extip:$(wget -qO- eth0.me)
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

```

```
sudo tee /etc/systemd/system/0gchaind.service > /dev/null <<EOF
[Unit]
Description=0gchaind Node Service
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/aristotle-used
ExecStart=$(which 0gchaind) start \
--rpc.laddr tcp://0.0.0.0:${OG_PORT}657 \
--chaincfg.chain-spec mainnet \
--chaincfg.kzg.trusted-setup-path $HOME/aristotle-used/kzg-trusted-setup.json \
--chaincfg.engine.jwt-secret-path $HOME/aristotle-used/jwt.hex \
--chaincfg.kzg.implementation=crate-crypto/go-kzg-4844 \
--chaincfg.block-store-service.enabled \
--chaincfg.node-api.enabled \
--chaincfg.node-api.logging \
--chaincfg.node-api.address 0.0.0.0:${OG_PORT}500 \
--chaincfg.engine.rpc-dial-url http://localhost:${OG_PORT}551 \
--pruning=nothing \
--p2p.seeds 9f78a3abe4be4462cdc2c3cae3c49a6a88ec16bc@8.221.140.134:26656 \
--p2p.external_address $(wget -qO- eth0.me):${OG_PORT}656 \
--home $HOME/.0gchaind/0g-home/0gchaind-home \
--chaincfg.restaking.enabled \
--chaincfg.restaking.symbiotic-rpc-dial-url https://ethereum-holesky-rpc.publicnode.com \
--chaincfg.restaking.symbiotic-get-logs-block-range 1
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable 0ggeth 0gchaind
sudo systemctl restart 0gchaind 0ggeth
```
# log monitor
```
sudo journalctl -u 0gchaind -u 0ggeth -f --no-hostname -o cat
```
