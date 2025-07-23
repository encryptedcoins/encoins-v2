# ENCOINS v2 project

## Relay server installation

This is a guide to run a relay server for ENCOINS v2. The server was developed by zkFold and is also known as the UTxO Accumulator server.

### Prerequisites

The server requires several cryptographic libraries to be installed on your machine (same as for running `cardano-node`):

1. **Install LIBSODIUM**:
   ```bash
   git clone https://github.com/input-output-hk/libsodium
   cd libsodium
   git checkout dbb48cc
   ./autogen.sh
   ./configure
   make
   sudo make install
   sudo ldconfig
   ```

2. **Install SECP256K1**:
   ```bash
   git clone https://github.com/bitcoin-core/secp256k1
   cd secp256k1
   git checkout ac83be33d0956faf6b7f61a60ab524ef7d6a473a
   ./autogen.sh
   ./configure --prefix=/usr --enable-module-schnorrsig --enable-experimental
   make
   sudo make install
   sudo ldconfig
   # Create symlink for version compatibility if needed
   if [ -f /usr/lib/libsecp256k1.so.0 ] && [ ! -f /usr/lib/libsecp256k1.so.2 ]; then
     sudo ln -sf /usr/lib/libsecp256k1.so.0 /usr/lib/libsecp256k1.so.2
   fi
   ```

3. **Install BLST**:
   ```bash
   BLST_VERSION='v0.3.11'
   git clone --depth 1 --branch ${BLST_VERSION} https://github.com/supranational/blst
   cd blst
   ./build.sh
   # Install library files
   sudo cp bindings/blst_aux.h bindings/blst.h bindings/blst.hpp /usr/local/include/
   sudo cp libblst.a /usr/local/lib/
   # Create pkg-config file
   cat > libblst.pc << EOF
   prefix=/usr/local
   exec_prefix=\${prefix}
   libdir=\${exec_prefix}/lib
   includedir=\${prefix}/include

   Name: libblst
   Description: Multilingual BLS12-381 signature library
   URL: https://github.com/supranational/blst
   Version: ${BLST_VERSION#v}
   Cflags: -I\${includedir}
   Libs: -L\${libdir} -lblst
   EOF
   sudo cp libblst.pc /usr/local/lib/pkgconfig/
   sudo chmod u=rw,go=r /usr/local/lib/libblst.a /usr/local/lib/pkgconfig/libblst.pc /usr/local/include/blst*
   ```

### Installation steps

1. **Create a new directory for the server**:
   ```bash
   mkdir -p utxo-accumulator-server
   cd utxo-accumulator-server
   ```

2. **Download the setup script**:
   ```bash
   curl -L "https://raw.githubusercontent.com/zkFold/utxo-accumulator-server/main/setup.sh" -o setup.sh
   chmod +x setup.sh
   ```

3. **Run the setup script**:
   ```bash
   ./setup.sh
   ```

4. **Configure the server**:
   After setup is complete, you need to manually update the configuration files:

   a. **Choose a configuration file** to edit (e.g., `config/100ada.yaml` for a 100 ADA pool, etc.)

   b. **Update the following required fields**:

   - **`port`**: Set your desired port number (it must be open on your machine)
   - **`maestroToken`**: Update with your Maestro API token
   - **`wallet.contents.mnemonic`**: Replace with your 24-word wallet mnemonic phrase
   - **`networkId`**: Ensure it matches your target network (`mainnet` or `preprod`)

   **Example configuration sections to update**:
   ```yaml
   port: 8082                          # Change to your desired port
   networkId: preprod                  # or "mainnet"
   coreProvider:
     maestroToken: YOUR_MAESTRO_TOKEN  # Replace with your token
   wallet:
     contents:
       mnemonic:
         - word1    # Replace with your actual
         - word2    # 24-word mnemonic phrase
         # ... continue for all 24 words
   ```

5. **Verify the server installation**:
   - Check that the binary `utxo-accumulator-server` exists and is executable
   - Verify all required configuration files and directories are present:
     - `config/` directory with YAML configuration files
     - `crs.json` (common reference string for zero-knowledge proofs)
     - `database/cache.json` (local database cache file)
     - `web/openapi/api.json` (API documentation)

6. **Test the server**:
   This step should output the server's command list:
   ```bash
   ./utxo-accumulator-server --help
   ```
7. **Launch the server**:
   Use the following command to run the server for a 100 ADA pool:
   ```bash
   ./utxo-accumulator-server run -c config/100ada.yaml --clean-db
   ```
