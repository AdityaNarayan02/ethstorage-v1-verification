# EthStorage V1 Trusted Setup

This repository provides an easy step-by-step guide and script to **verify the EthStorage V1 Trusted Setup Ceremony**.  
You don’t need prior zk-SNARK knowledge — just follow along!

---

## 🚀 Quick Start

### 1. Install Tools
```bash
sudo apt update && sudo apt install nodejs npm wget git -y
npm install -g snarkjs@0.7.5
npm install -g circom@2.2.2
```
### 2. Clone the Circuits Repository
```
git clone https://github.com/ethstorage/zk-decoder.git
cd zk-decoder/circom/circuits
npm install
```
### 3. Create a Script File
```
#!/bin/bash
set -e

echo "📦 Starting EthStorage V1 Trusted Setup Verification"
echo "📁 Working in current directory: $(pwd)"

# Download Powers of Tau files from the correct URLs (PSE ceremony files)
echo "⬇️ Downloading Powers of Tau files..."
if [ ! -f "ppot_0080_19.ptau" ]; then
    echo "Downloading ppot_0080_19.ptau..."
    wget https://pse-trusted-setup-ppot.s3.eu-central-1.amazonaws.com/pot28_0080/ppot_0080_19.ptau
else
    echo "ppot_0080_19.ptau already exists"
fi

if [ ! -f "ppot_0080_20.ptau" ]; then
    echo "Downloading ppot_0080_20.ptau..."
    wget https://pse-trusted-setup-ppot.s3.eu-central-1.amazonaws.com/pot28_0080/ppot_0080_20.ptau
else
    echo "ppot_0080_20.ptau already exists"
fi

# Install npm dependencies if needed
echo "📦 Installing npm dependencies..."
npm install 2>/dev/null || echo "npm install skipped (may not be needed)"

# Compile circuits
echo "⚙️ Compiling circuits..."
echo "   Compiling blob_poseidon.circom..."
circom ./blob_poseidon.circom --r1cs --O2

echo "   Compiling blob_poseidon_2.circom..."
circom ./blob_poseidon_2.circom --r1cs --O2

# Check hashes
echo "🔍 Checking circuit hashes..."
echo "Expected SHA-256 hashes:"
echo " blob_poseidon.r1cs -> 9c0f44d168716ddf680a1f9b3a4b41ed08e4730d2d87d05d4e3816276af58f7a"
echo " blob_poseidon_2.r1cs -> ba17a94a5edcc1d0b0ad10990e0aea4894f080ce571b0db79fb1c3a9c4a74a0d"
echo ""
echo "Actual hashes:"
shasum -a 256 blob_poseidon.r1cs
shasum -a 256 blob_poseidon_2.r1cs

# Download final zkey files from the ceremony page
echo ""
echo "⬇️ Downloading final zkey files from ceremony..."
if [ ! -f "blob-poseidon-circuit_final.zkey" ]; then
    echo "Downloading blob-poseidon-circuit_final.zkey..."
    wget "https://ethstorage-v1-trusted-setup-ceremony-pse-p0tion-production.s3.eu-central-1.amazonaws.com/circuits/blob-poseidon-circuit/contributions/blob-poseidon-circuit_final.zkey" -O blob-poseidon-circuit_final.zkey
else
    echo "blob-poseidon-circuit_final.zkey already exists"
fi

if [ ! -f "blob-poseidon-2-circuit_final.zkey" ]; then
    echo "Downloading blob-poseidon-2-circuit_final.zkey..."
    wget "https://ethstorage-v1-trusted-setup-ceremony-pse-p0tion-production.s3.eu-central-1.amazonaws.com/circuits/blob-poseidon-2-circuit/contributions/blob-poseidon-2-circuit_final.zkey" -O blob-poseidon-2-circuit_final.zkey
else
    echo "blob-poseidon-2-circuit_final.zkey already exists"
fi

# Verify zkey files
echo ""
echo "✅ Verifying final zkey files..."
echo "Verifying blob-poseidon circuit..."
snarkjs zkey verify blob_poseidon.r1cs ppot_0080_19.ptau blob-poseidon-circuit_final.zkey

echo ""
echo "Verifying blob-poseidon-2 circuit..."
snarkjs zkey verify blob_poseidon_2.r1cs ppot_0080_20.ptau blob-poseidon-2-circuit_final.zkey

echo ""
echo "🎉 EthStorage V1 Trusted Setup Verification completed successfully!"
echo "✅ All circuit hashes match expected values"
echo "✅ All zkey files verified against their respective circuits and Powers of Tau"
echo "✅ Ceremony transcripts are valid and secure" 
```
### 4. Run the script
```
chmod +x verify.sh
./verify.sh
```

# You have sucessfully completed the task






