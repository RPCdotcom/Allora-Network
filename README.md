# Allora-Network

![image](https://github.com/user-attachments/assets/8a9f195e-9ed1-4cdb-b2ea-9a1037b77091)

> Requirements : 

> Operating System: Ubuntu 22.04

> CPU: Minimum of 1/2 core.

> Memory: 2 to 4 GB.

> Storage: SSD or NVMe with at least 5GB of space.

 # Install Update / Packages : 

```console
sudo apt update -y && sudo apt upgrade -y
```

```console
sudo apt install ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev curl git wget make jq build-essential pkg-config lsb-release libssl-dev libreadline-dev libffi-dev gcc screen unzip lz4 -y
```

# Install Python3

```console
sudo apt install python3
```

```console
sudo apt install python3-pip
```

# Install Docker

```console
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```console
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```console
sudo apt-get update
```

```console
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

```console
docker version
```

# Install Docker-Compose 

```console
VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)
```

```console
curl -L "https://github.com/docker/compose/releases/download/"$VER"/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```


```console
chmod +x /usr/local/bin/docker-compose
```


```console
docker-compose --version
```

# Docker Permission to User

```console
sudo groupadd docker
```

```console
sudo usermod -aG docker $USER
```

# Install Go 

```console
cd $HOME && \
ver="1.21.3" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Install Allorad / Wallet

```console
git clone https://github.com/allora-network/allora-chain.git
```

```console
cd allora-chain && make all
```

```console
allorad version
```

# Add Wallet or Crate Wallet

> If you have Keplr Wallet , you can import the wallet.

> Recover your wallet with seed-phrase  :

```console
allorad keys add testkey --recover
```
> Crate a New Wallet : 

```console
allorad keys add testkey
```
# Faucet 

> Connect to Allora for found your Allora network adress : https://app.allora.network/points/campaigns

> You Can Add Allora Nework to Wallet - here : https://explorer.testnet-1.testnet.allora.network/allora-testnet-1

> uAllo faucet : https://faucet.testnet-1.testnet.allora.network/

# Install Worker : 

> Install : 

```console
git clone https://github.com/allora-network/basic-coin-prediction-node
```

```console
cd basic-coin-prediction-node
```

```console
mkdir worker-data
```

```console
mkdir head-data
```

# Give Certain Permissions :

```console
sudo chmod -R 777 worker-data
```

```console
sudo chmod -R 777 head-data
```

# Crate Head Keys : 

```console
sudo docker run -it --entrypoint=bash -v ./head-data:/data alloranetwork/allora-inference-base:latest -c "mkdir -p /data/keys && (cd /data/keys && allora-keys)"
```

```console
sudo docker run -it --entrypoint=bash -v ./worker-data:/data alloranetwork/allora-inference-base:latest -c "mkdir -p /data/keys && (cd /data/keys && allora-keys)"
```

# Copy The Head - Id : 

> This is your head ID, you will need it in the next step

```console
cat head-data/keys/identity
```

![image](https://github.com/user-attachments/assets/d498fff0-b855-465c-8542-8d39feaa5410)

# Connect to Allora Chain : 

> Delete and crate a new docker-compose.yml file : 

```console
rm -rf docker-compose.yml && nano docker-compose.yml
```

> Copy And Paste this code : 


> !!!!! Change head-id and Wallet_Seed_Phares ( 24 Wallet Key ) !!!!

```console
version: '3'

services:
  inference:
    container_name: inference-basic-eth-pred
    build:
      context: .
    command: python -u /app/app.py
    ports:
      - "8000:8000"
    networks:
      eth-model-local:
        aliases:
          - inference
        ipv4_address: 172.22.0.4
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/inference/ETH"]
      interval: 10s
      timeout: 10s
      retries: 12
    volumes:
      - ./inference-data:/app/data

  updater:
    container_name: updater-basic-eth-pred
    build: .
    environment:
      - INFERENCE_API_ADDRESS=http://inference:8000
    command: >
      sh -c "
      while true; do
        python -u /app/update_app.py;
        sleep 24h;
      done
      "
    depends_on:
      inference:
        condition: service_healthy
    networks:
      eth-model-local:
        aliases:
          - updater
        ipv4_address: 172.22.0.5

  worker:
    container_name: worker-basic-eth-pred
    environment:
      - INFERENCE_API_ADDRESS=http://inference:8000
      - HOME=/data
    build:
      context: .
      dockerfile: Dockerfile_b7s
    entrypoint:
      - "/bin/bash"
      - "-c"
      - |
        if [ ! -f /data/keys/priv.bin ]; then
          echo "Generating new private keys..."
          mkdir -p /data/keys
          cd /data/keys
          allora-keys
        fi
        # Change boot-nodes below to the key advertised by your head
        allora-node --role=worker --peer-db=/data/peerdb --function-db=/data/function-db \
          --runtime-path=/app/runtime --runtime-cli=bls-runtime --workspace=/data/workspace \
          --private-key=/data/keys/priv.bin --log-level=debug --port=9011 \
          --boot-nodes=/ip4/172.22.0.100/tcp/9010/p2p/head-id \
          --topic=allora-topic-1-worker \
          --allora-chain-key-name=testkey \
          --allora-chain-restore-mnemonic='WALLET_SEED_PHRASE' \
          --allora-node-rpc-address=https://allora-rpc.testnet-1.testnet.allora.network/ \
          --allora-chain-topic-id=1
    volumes:
      - ./worker-data:/data
    working_dir: /data
    depends_on:
      - inference
      - head
    networks:
      eth-model-local:
        aliases:
          - worker
        ipv4_address: 172.22.0.10

  head:
    container_name: head-basic-eth-pred
    image: alloranetwork/allora-inference-base-head:latest
    environment:
      - HOME=/data
    entrypoint:
      - "/bin/bash"
      - "-c"
      - |
        if [ ! -f /data/keys/priv.bin ]; then
          echo "Generating new private keys..."
          mkdir -p /data/keys
          cd /data/keys
          allora-keys
        fi
        allora-node --role=head --peer-db=/data/peerdb --function-db=/data/function-db  \
          --runtime-path=/app/runtime --runtime-cli=bls-runtime --workspace=/data/workspace \
          --private-key=/data/keys/priv.bin --log-level=debug --port=9010 --rest-api=:6000
    ports:
      - "6000:6000"
    volumes:
      - ./head-data:/data
    working_dir: /data
    networks:
      eth-model-local:
        aliases:
          - head
        ipv4_address: 172.22.0.100


networks:
  eth-model-local:
    driver: bridge
    ipam:
      config:
        - subnet: 172.22.0.0/24

volumes:
  inference-data:
  worker-data:
  head-data:
```

> CTRL X - CTRL Y - Enter.

```console
docker compose build
docker compose up -d
```
