# Omni-Network-Validator-Node

### Prerequisites

Before starting, make sure you meet the following requirements:

1. **Server Requirements:**
   - **OS:** Ubuntu 20.04+ (Recommended)
   - **CPU:** 4+ cores
   - **Memory:** 8GB+ RAM
   - **Storage:** SSD with 250GB+ free disk space
   - **Network:** 1 Gbps network connection

2. **Install Dependencies:**
   - Install **Docker** and **Docker Compose** (Omni uses Docker for container management).

   ```bash
   sudo apt-get update
   sudo apt-get install -y curl apt-transport-https ca-certificates software-properties-common
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   sudo apt update
   sudo apt install -y docker-ce docker-compose
   ```

3. **Create a Keypair:**
   You will need an Omni Network keypair for signing your transactions.

   - Install Omni CLI:
     ```bash
     curl -LO https://github.com/OmniNetwork/cli/releases/download/v1.0.0/omni-linux-amd64
     chmod +x omni-linux-amd64
     sudo mv omni-linux-amd64 /usr/local/bin/omni
     ```

   - Generate keypair:
     ```bash
     omni key generate --output-keyfile omni_key.json
     ```

   - Back up your keyfile securely! This is your identity on the network.

---

### Step 1: Clone the Validator Repository

To start setting up your validator node, clone the Omni Network's validator repository.

```bash
git clone https://github.com/OmniNetwork/validator.git
cd validator
```

### Step 2: Configure the Validator Node

In the cloned repository, there is a configuration file `config.toml` which needs to be customized.

1. **Edit Configuration File**:
   Open the configuration file in your favorite text editor:

   ```bash
   nano config.toml
   ```

   2. Modify the following fields:
   
   - **`moniker`**: Set your validator’s moniker. This is how your validator will be identified.
   
   - **`chain_id`**: Make sure the correct chain ID is specified. Refer to the network documentation for the current chain ID.

   - **`keyring_backend`**: Set to `"file"`.

   - **`seeds`**: This is the list of seed nodes to connect to the network. Use the seed nodes provided by Omni:
     ```
     seeds = "node1.omni.network:26656,node2.omni.network:26656"
     ```

   - **`gas_prices`**: Set the desired gas price for transactions. For example:
     ```
     gas_prices = "0.025uomni"
     ```

---

### Step 3: Initialize the Validator Node

Once the configuration is set, initialize the validator node:

```bash
omni init [validator-name] --chain-id=[chain-id]
```

This will create the necessary configuration files and directories for your validator node.

---

### Step 4: Start the Validator Node

To start your validator node, you’ll use Docker Compose. First, check the `docker-compose.yml` file to ensure it's set up correctly. Then, run:

```bash
docker-compose up -d
```

This will start your validator node in the background.

### Step 5: Check Node Sync Status

You need to ensure that your validator node is fully synced with the network before becoming active. To check the sync status:

```bash
docker logs -f omni-validator
```

Look for logs that indicate the node is fully synced, such as:

```
INFO: Fast sync complete
```

---

### Step 6: Create the Validator

Once your node is synced, it’s time to create your validator on the Omni Network.

```bash
omni tx staking create-validator \
  --amount=1000000uomni \
  --pubkey=$(omni tendermint show-validator) \
  --moniker=[your-validator-name] \
  --chain-id=[chain-id] \
  --commission-rate=0.10 \
  --commission-max-rate=0.20 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation=1 \
  --gas=auto \
  --from=[your-key-name]
```

**Explanation of Flags:**

- **`--amount`**: Amount of Omni tokens to bond (this example uses 1,000,000 `uomni`).
- **`--pubkey`**: Public key of your validator node, obtained with `omni tendermint show-validator`.
- **`--commission-rate`**: Initial commission rate for delegators.
- **`--commission-max-rate`**: Maximum commission rate.
- **`--commission-max-change-rate`**: Maximum daily change in commission rate.
- **`--min-self-delegation`**: Minimum amount of tokens you’ll self-delegate.

If successful, you should see a transaction hash confirming the validator creation.

---

### Step 7: Monitor and Maintain Your Validator

1. **Check Validator Status**:

   To verify your validator is running:

   ```bash
   omni query staking validator $(omni tendermint show-address)
   ```

2. **Get Logs**:

   To monitor logs, use the following command:

   ```bash
   docker logs -f omni-validator
   ```

3. **Update and Restart Node**:

   If there are updates to the network, stop your node, pull the latest changes from the repo, and restart:

   ```bash
   docker-compose down
   git pull origin main
   docker-compose up -d
   ```
