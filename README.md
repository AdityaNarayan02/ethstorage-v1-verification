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

# Color codes for better output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Function to print colored output
print_status() {
    echo -e "${BLUE}$1${NC}"
}

print_success() {
    echo -e "${GREEN}$1${NC}"
}

print_warning() {
    echo -e "${YELLOW}$1${NC}"
}

print_error() {
    echo -e "${RED}$1${NC}"
}

# Function to check if command exists
command_exists() {
    command -v "$1" >/dev/null 2>&1
}

# Function to verify file hash
verify_hash() {
    local file="$1"
    local expected_hash="$2"
    local actual_hash
    
    if [[ ! -f "$file" ]]; then
        print_error "❌ File $file not found"
        return 1
    fi
    
    actual_hash=$(shasum -a 256 "$file" | cut -d' ' -f1)
    
    if [[ "$actual_hash" == "$expected_hash" ]]; then
        print_success "✅ $file hash verified: $actual_hash"
        return 0
    else
        print_error "❌ Hash mismatch for $file"
        print_error "   Expected: $expected_hash"
        print_error "   Actual:   $actual_hash"
        return 1
    fi
}

# Function to download file with progress and verification
download_file() {
    local url="$1"
    local output="$2"
    local description="$3"
    
    if [[ -f "$output" ]]; then
        print_warning "⚠️  $output already exists, skipping download"
        return 0
    fi
    
    print_status "⬇️  Downloading $description..."
    if command_exists curl; then
        curl -L --progress-bar "$url" -o "$output"
    elif command_exists wget; then
        wget --progress=bar "$url" -O "$output"
    else
        print_error "❌ Neither curl nor wget found. Please install one of them."
        exit 1
    fi
    
    if [[ $? -eq 0 ]]; then
        print_success "✅ Downloaded $description successfully"
    else
        print_error "❌ Failed to download $description"
        exit 1
    fi
}

# Main execution starts here
print_status "📦 Starting EthStorage V1 Trusted Setup Verification"

# Check for required tools
required_tools=("circom" "snarkjs" "shasum")
missing_tools=()

for tool in "${required_tools[@]}"; do
    if ! command_exists "$tool"; then
        missing_tools+=("$tool")
    fi
done

if [[ ${#missing_tools[@]} -gt 0 ]]; then
    print_error "❌ Missing required tools: ${missing_tools[*]}"
    print_error "Please install the missing tools and try again."
    exit 1
fi

# Create and navigate to circuits directory
CIRCUITS_DIR="zk-decoder/circom/circuits"
print_status "📁 Creating and navigating to $CIRCUITS_DIR"
mkdir -p "$CIRCUITS_DIR"
cd "$CIRCUITS_DIR"

# Download Powers of Tau files
print_status "⬇️ Downloading Powers of Tau files..."
download_file "https://hermez.s3-eu-west-1.amazonaws.com/ppot_0080_19.ptau" "ppot_0080_19.ptau" "Powers of Tau 19"
download_file "https://hermez.s3-eu-west-1.amazonaws.com/ppot_0080_20.ptau" "ppot_0080_20.ptau" "Powers of Tau 20"

# Download circuit files if they don't exist
if [[ ! -f "blob_poseidon.circom" ]]; then
    print_status "⬇️ Downloading blob_poseidon.circom..."
    download_file "https://raw.githubusercontent.com/ethstorage/zk-decoder/main/circom/circuits/blob_poseidon.circom" "blob_poseidon.circom" "blob_poseidon circuit"
fi

if [[ ! -f "blob_poseidon_2.circom" ]]; then
    print_status "⬇️ Downloading blob_poseidon_2.circom..."
    download_file "https://raw.githubusercontent.com/ethstorage/zk-decoder/main/circom/circuits/blob_poseidon_2.circom" "blob_poseidon_2.circom" "blob_poseidon_2 circuit"
fi

# Compile circuits
print_status "⚙️  Compiling circuits..."
print_status "   Compiling blob_poseidon.circom..."
circom ./blob_poseidon.circom --r1cs --O2 -l ./

print_status "   Compiling blob_poseidon_2.circom..."
circom ./blob_poseidon_2.circom --r1cs --O2 -l ./

print_success "✅ Circuit compilation completed"

# Verify circuit hashes
print_status "🔍 Verifying circuit hashes..."

BLOB_POSEIDON_HASH="9c0f44d168716ddf680a1f9b3a4b41ed08e4730d2d87d05d4e3816276af58f7a"
BLOB_POSEIDON_2_HASH="ba17a94a5edcc1d0b0ad10990e0aea4894f080ce571b0db79fb1c3a9c4a74a0d"

verify_hash "blob_poseidon.r1cs" "$BLOB_POSEIDON_HASH" || exit 1
verify_hash "blob_poseidon_2.r1cs" "$BLOB_POSEIDON_2_HASH" || exit 1

# Download final zkey files
print_status "⬇️ Downloading final zkey files..."
download_file "https://ethstorage-v1-trusted-setup-ceremony-pse-p0tion-production.s3.eu-central-1.amazonaws.com/circuits/blob-poseidon-circuit/contributions/blob-poseidon-circuit_final.zkey" "blob-poseidon-circuit_final.zkey" "blob-poseidon final zkey"
download_file "https://ethstorage-v1-trusted-setup-ceremony-pse-p0tion-production.s3.eu-central-1.amazonaws.com/circuits/blob-poseidon-2-circuit/contributions/blob-poseidon-2-circuit_final.zkey" "blob-poseidon-2-circuit_final.zkey" "blob-poseidon-2 final zkey"

# Verify zkeys
print_status "✅ Verifying final zkeys..."
print_status "   Verifying blob-poseidon zkey..."
if snarkjs zkey verify blob_poseidon.r1cs ppot_0080_19.ptau blob-poseidon-circuit_final.zkey; then
    print_success "✅ blob-poseidon zkey verification passed"
else
    print_error "❌ blob-poseidon zkey verification failed"
    exit 1
fi

print_status "   Verifying blob-poseidon-2 zkey..."
if snarkjs zkey verify blob_poseidon_2.r1cs ppot_0080_20.ptau blob-poseidon-2-circuit_final.zkey; then
    print_success "✅ blob-poseidon-2 zkey verification passed"
else
    print_error "❌ blob-poseidon-2 zkey verification failed"
    exit 1
fi

print_success "🎉 EthStorage V1 Trusted Setup Verification completed successfully!"
print_status "📋 Summary:"
print_status "   - Powers of Tau files downloaded and ready"
print_status "   - Circuit files compiled successfully"
print_status "   - Circuit hashes verified"
print_status "   - Final zkey files downloaded and verified"
print_status ""
print_success "🔐 Your trusted setup verification is complete and secure!"
```
### 4. Run the script
```
chmod +x verify.sh
./verify.sh
```

# You have sucessfully completed the task






