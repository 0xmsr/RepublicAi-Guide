# 🚀 Republic AI Node Set up - Monitoring Guide
Dokumentasi lengkap untuk instalasi, konfigurasi, dan pemantauan Node Republic AI Testnet.

[![Network](https://img.shields.io/badge/Network-republic_ai_testnet-blue.svg)](https://github.com/republic-ai)
[![Go Version](https://img.shields.io/badge/Go-1.21.6+-00ADD8.svg?style=flat&logo=go)](https://go.dev/)

## 📋 Prasyaratan Sistem (Hardware Requirements)
##### OS: Ubuntu 22.04 LTS atau lebih tinggi
##### Spesifikasi Minimum: 4 vCPU, 8GB RAM, 100GB Disk.

---

## 1. Instalasi Environment
**Lakukan pembaruan sistem dan pasang bahasa pemrograman Go.**

```bash
# Update sistem
sudo apt update && sudo apt upgrade -y
sudo apt install curl git jq lz4 build-essential -y

# Instalasi Go (v1.21.6)
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc
source $HOME/.bashrc
```

---

## 2. Build & Konfigurasi Node
**Lakukan build binari dari source code resmi.**

```bash
git clone https://github.com/republic-ai/republic-node.git
cd republic-node
make install

# Inisialisasi (Ganti <MONIKER> dengan nama node kamu)
republicd init <MONIKER> --chain-id republic_20241-1

# Genesis & Peers
curl -L https://raw.githubusercontent.com/republic-ai/republic-node/main/genesis.json > $HOME/.republic/config/genesis.json
PEERS="d512a97cf1e4ee4544d678864f1d4ed9a6f3b063@152.42.203.111:26656,e1250280f5572e81eddf4401a742c525f0a0d912@157.245.197.66:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.republic/config/config.toml
```

---

## 3. Manajemen Wallet
Pilih metode yang sesuai untuk akun kamu.

**Restore Wallet:** 
```bash
republicd keys add NamaWallet --recover
```
**Cek Saldo:** 
```bash
republicd q bank balances $(republicd keys show NamaWallet -a)
```

---

## 4. Menjalankan Node & Validator
Jalankan node di background atau terminal terpisah.

```bash
# Start Node
republicd start

# Create Validator (Pastikan sudah Sync)
republicd tx staking create-validator \
  --amount=1000000arai \
  --pubkey=$(republicd comet show-validator) \
  --moniker="<MONIKER>" \
  --chain-id=republic_20241-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=NamaWallet \
  --gas-prices="0.1arai" \
  --gas="auto" \
  --gas-adjustment="1.5" \
  -y
```

---

## 5. Script Monitoring Otomatis
Script ini memantau Reward dan Index Offset (absen/tidaknya node) secara real-time setiap 60 detik.

Cara Penggunaan:
Simpan kode berikut ke dalam file gacor.sh atau Paste langsung ke terminal:

```bash
#!/bin/bash
MY_ADDR="rai..."
VAL_ADDR="raivaloper..."

while true; do
  echo "--------------------------------------------------------"
  echo "🕒 Waktu: $(date)"
  
  # Fetch Reward (Anti-JQ Error)
  REWARD=$(republicd q distribution rewards $MY_ADDR 2>/dev/null | grep -A 1 "total:" | grep "arai" | awk '{print $2}' | sed 's/arai//')
  
  # Fetch Offset
  OFFSET=$(republicd q slashing signing-info $(republicd comet show-validator) 2>/dev/null | grep "index_offset" | awk -F'"' '{print $2}')
  
  echo "💰 Reward       : ${REWARD:-"0"} arai"
  echo "📈 Index Offset : ${OFFSET:-"0"}"
  echo "🛡️ Status       : BOND_STATUS_BONDED"
  echo "--------------------------------------------------------"
  sleep 60
done
```
#### Ganti bagian dengan wallet address dan validator address punya kamu
###### MY_ADDR="rai..."
###### VAL_ADDR="rai..."

### Run gacor.sh: 
```bash
gacor.sh
```
---

## Tabel Perintah Penting 

| Task | Command |
| :--- | :--- |
| **Klaim Reward** | `republicd tx distribution withdraw-all-rewards --from NamaWallet --gas-prices 0.1arai -y` |
| **Cek Reward Pending** | `republicd q distribution rewards $(republicd keys show NamaWallet -a)` |
| **Delegate** | `republicd tx staking delegate <VALOPER_ADDR> <JUMLAH>arai --from NamaWallet --gas-prices 1000000000arai --gas auto --gas-adjustment 1.5 -y` |
| **Cek Sync** | `republicd status 2>&1 \| jq .SyncInfo` |
| **Unjail Node** | `republicd tx slashing unjail --from NamaWallet --gas-prices 1000000000arai --gas auto --gas-adjustment 1.5 -y` |
| **Signing Info** | `republicd q slashing signing-info $(republicd comet show-validator)` |
| **Status Val** | `republicd q staking validator $(republicd keys show NamaWallet --bech val -a)` |

##### Penting untuk memahami konversi satuan agar tidak salah saat melakukan delegasi atau transaksi:
* **1 RAI** = $1.000.000.000.000.000.000$ **arai** ($10^{18}$)
* **arai** adalah satuan terkecil (minimal unit) yang digunakan di dalam sistem/CLI.

---

*Thank you for reading!* – **0xmsr**






