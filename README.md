# ethstorage-v1-verification

# EthStorage V1 Trusted Setup Verification

This repository provides an easy step-by-step guide and script to **verify the EthStorage V1 Trusted Setup Ceremony**.  
You don’t need prior zk-SNARK knowledge — just follow along!

---

## 🚀 Quick Start

### 1. Install Tools
```bash
sudo apt update && sudo apt install nodejs npm wget git -y
npm install -g snarkjs@0.7.5
npm install -g circom2
```
### 2. Clone the Circuits Repository
```bash
git clone https://github.com/ethstorage/zk-decoder.git
cd zk-decoder/circom/circuits
npm install
```
### 3. Download the script
```
wget https://github.com/AdityaNarayan02/ethstorage-v1-verification/blob/246592f276b645f2f892c6642edc0673641ae478/verify.sh
chmod +x verify.sh
```
### 4. Run the verification Script
```
./verify.sh
```
### 5. ✅ Expected Results
At the end, you should see:
```
snarkJS: ZKey Ok!
```
for both circuits.

That means you have independently verified the EthStorage Trusted Setup 🎉
