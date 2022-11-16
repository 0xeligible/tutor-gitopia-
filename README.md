<p style="font-size:14px" align="right">
<a href=" target="_blank">Join our telegram <img src="https://user-images.githubusercontent.com/50621007/183283867-56b4d69f-bc6e-4939-b00a-72aa019d1aea.png" width="30"/></a>
<a href=" target="_blank">Visit our website <img src="https://user-images.githubusercontent.com/38981255/184068977-2d456b1a-9b50-4b75-a0a7-4909a7c78991.png" width="30"/></a>
</p>

<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/38981255/201691739-df61f26e-5c05-46b5-8ce6-0981a6615d60.PNG">
</p>

#  penyiapan node gitopia untuk testnet — gitopia-janus-testnet-2

Dokumentasi resmi:
> - [Petunjuk penyiapan validator](https://docs.gitopia.com/installation/index.html)

Explorer :
> - [Explorer Checker](https://gitopia.explorers.guru/)

##  Persyaratan Perangkat Keras
Seperti rantai Cosmos-SDK lainnya, persyaratan perangkat kerasnya cukup sederhana.

###  Persyaratan Perangkat Keras Minimum
 - 4x CPU; semakin cepat kecepatan jam semakin baik
 -RAM 8GB
 - Penyimpanan 100GB (SSD atau NVME)
 - Koneksi Internet permanen (lalu lintas akan minimal selama testnet; 10Mbps akan banyak - untuk produksi diharapkan setidaknya 100Mbps)

###  Persyaratan Perangkat Keras yang Direkomendasikan
 - 8x CPU; semakin cepat kecepatan jam semakin baik
 -RAM 64GB
 - Penyimpanan 1TB (SSD atau NVME)
 - Koneksi Internet permanen (lalu lintas akan minimal selama testnet; 10Mbps akan banyak - untuk produksi diharapkan setidaknya 100Mbps)

##  Siapkan gitopia fullnode Anda 
###  Opsi 1 (otomatis)
Anda dapat mengatur gitopia fullnode Anda dalam beberapa menit dengan menggunakan skrip otomatis di bawah ini. Ini akan meminta Anda untuk memasukkan nama simpul validator Anda!
```
wget -O gitopia.sh https://raw.githubusercontent.com/bangpateng/gitopia/main/gitopia.sh && chmod +x gitopia.sh && ./gitopia.sh
```

##  Pasang instalasi

Saat instalasi selesai, silakan muat variabel ke dalam sistem
```
source $HOME/.bash_profile
```

Selanjutnya Anda harus memastikan validator Anda menyinkronkan blok. Anda dapat menggunakan perintah di bawah ini untuk memeriksa status sinkronisasi
```
gitopiad status 2>&1 | jq .SyncInfo
```
### Create wallet
Membuat Wallet baru dan Backup Address dan Pharse Anda
```
gitopiad keys add $WALLET
```

(OPTIONAL) Jika Anda Mempunyai Wallet Sebelumnya, Silahkan Import Pharse
```
gitopiad keys add $WALLET --recover
```

To get current list of wallets
```
gitopiad keys list
```

### Save Info Wallet
Add wallet and valoper address into variables 
```
GITOPIA_WALLET_ADDRESS=$(gitopiad keys show $WALLET -a)
GITOPIA_VALOPER_ADDRESS=$(gitopiad keys show $WALLET --bech val -a)
echo 'export GITOPIA_WALLET_ADDRESS='${GITOPIA_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export GITOPIA_VALOPER_ADDRESS='${GITOPIA_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Fund your wallet

- Import Validator Pharse Wallet Kalian ke Kepler
- Open Web : https://gitopia.com/
- Connect Wallet
- Get 10 Tlore Done

### SNAPSHOOT (Form KjNodes) Buat Loncat Ke Block Saat Ini

Stop Layanan dan Reset

```
sudo systemctl stop gitopiad
cp $HOME/.gitopia/data/priv_validator_state.json $HOME/.gitopia/priv_validator_state.json.backup
rm -rf $HOME/.gitopia/data
```
Download Snapshoot Terbaru (Size File 880 MB)
```
curl -L https://snapshots.kjnodes.com/gitopia-testnet/snapshot_latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.gitopia
mv $HOME/.gitopia/priv_validator_state.json.backup $HOME/.gitopia/data/priv_validator_state.json
```
Muat Ulang Layanan dan Periksa Block
```
sudo systemctl start gitopiad && journalctl -u gitopiad -f --no-hostname -o cat
```

### Membuat validator
Before creating validator please make sure that you have at least 1 tlore (1 tlore is equal to 1000000 utlore) and your node is synchronized

To check your wallet balance:
```
gitopiad query bank balances $GITOPIA_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 
To create your validator run command below
```
gitopiad tx staking create-validator \
  --amount 1000000utlore \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(gitopiad tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $GITOPIA_CHAIN_ID
```
### Check your validator key
```
[[ $(gitopiad q staking validator $GITOPIA_VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key) = $(gitopiad status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```
