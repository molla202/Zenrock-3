```
sudo apt install liblz4-tool
systemctl stop zenrockd
cp $HOME/.zrchain/data/priv_validator_state.json $HOME/.zrchain/priv_validator_state.json.backup
cp $HOME/.zrchain/config/priv_validator_key.json $HOME/.zrchain/priv_validator_key.json.backup
zenrockd tendermint unsafe-reset-all --home $HOME/.zrchain --keep-addr-book
curl -L http://37.120.189.81/zenrock_testnet/zenrock_snap.tar.lz4 | tar -I lz4 -xf - -C $HOME/.zrchain
mv $HOME/.zrchain/priv_validator_state.json.backup $HOME/.zrchain/data/priv_validator_state.json
sudo systemctl daemon-reload
sudo systemctl restart zenrockd
sudo journalctl -u zenrockd.service -f --no-hostname -o cat
```
