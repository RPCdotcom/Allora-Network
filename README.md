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

