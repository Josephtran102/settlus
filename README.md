# settlus
set vars
```
echo "export WALLET="joseph"" >> $HOME/.bash_profile
echo "export MONIKER="JosephTran"" >> $HOME/.bash_profile
echo "export SETTLUS_CHAIN_ID="settlus_5372-2"" >> $HOME/.bash_profile
echo "export SETTLUS_PORT="52"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
# download binary
```
cd $HOME
wget -O settlusd [https://github.com/airchains-network/junction/releases/download/v0.1.0/junctiond](https://github.com/settlus/chain/releases/download/v0.5.4/settlus_0.5.4_Linux_amd64.tar.gz
chmod +x settlusd
mv settlusd $HOME/go/bin/
sudo mv ~/bin/settlusd /usr/local/bin/
sudo chmod +x /usr/local/bin/settlusd
```
# config and init app
```
settlusd init $MONIKER --chain-id $SETTLUS_CHAIN_ID 
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${SETTLUS_PORT}657\"|" $HOME/.settlus/config/client.toml
```
```
settlusd init $MONIKER \
--chain-id $SETTLUS_CHAIN_ID
```
# download genesis
```
wget -O $HOME/.settlus/config/genesis.json https://raw.githubusercontent.com/settlus/testnet/main/testnets/settlus_5372-2/genesis.json
```
# set seeds and peers
```
SEEDS="4ef5bf671434c6f1f49f043f8039e1f632645e9f@52.207.211.61:26656,2ded80262d11c4b9dae45a8f0a9d073ddfa93c00@43.202.2.83:26656,74d5cc68bb6532bc8bec5a89207a0c80c975c6b1@18.142.146.142:26656"
PEERS="4ef5bf671434c6f1f49f043f8039e1f632645e9f@52.207.211.61:26656,2ded80262d11c4b9dae45a8f0a9d073ddfa93c00@43.202.2.83:26656,74d5cc68bb6532bc8bec5a89207a0c80c975c6b1@18.142.146.142:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.settlus/config/config.toml
```
# set custom ports in config.toml file
```
sed -i.bak -e "
    s%:26658%:${SETTLUS_PORT}658%g;
    s%:26657%:${SETTLUS_PORT}657%g;
    s%:6060%:${SETTLUS_PORT}060%g;
    s%:26656%:${SETTLUS_PORT}656%g;
    s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${SETTLUS_PORT}656\"%;
    s%:26660%:${SETTLUS_PORT}660%g
" $HOME/.settlus/config/config.toml
```
# config pruning
```
sed -i -e "s/^pruning =.*/pruning = \"custom\"/" $HOME/.settlus/config/app.toml
sed -i -e "s/^pruning-keep-recent =.*/pruning-keep-recent = \"100\"/" $HOME/.settlus/config/app.toml
sed -i -e "s/^pruning-interval =.*/pruning-interval = \"50\"/" $HOME/.settlus/config/app.toml
```
# set minimum gas price, enable prometheus and disable indexing
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.001amf"|g' $HOME/.settlus/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.settlus/config/config.toml
sed -i -e "s/^indexer =.*/indexer = \"null\"/" $HOME/.settlus/config/config.toml
```
# create service file
```
sudo tee /etc/systemd/system/settlusd.service > /dev/null <<EOF
[Unit]
Description=Settlus node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.settlus
ExecStart=$(which settlusd) start --home $HOME/.settlus
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
# enable and start service
```
sudo systemctl daemon-reload
sudo systemctl enable settlusd
sudo systemctl restart settlusd && sudo journalctl -u junctiond -f
```
