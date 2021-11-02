# 1.Preparing Linux OS for node installation
# 1.1 We update packages and systems

***sudo apt update && sudo apt upgrade -y***

# 1.2 Installing all the necessary packages and components

***sudo apt install wget curl jq make git build-essential cmake gcc pkg-config libssl-dev -y***

For correct and safer work with Linux OS, it is recommended to create a separate user and not work as root. I often use the system's built-in administrator account. But if you decide to create a separate user and give him installation rights on the system, then this can be done using the following commands:

***adduser <_user name_>***

***usermod -aG sudo <_user name_>***

***su -l <_user name_>***

I will not create a new user and continue with the installation as root

# 2. Install GO
If the OS on the server has previously installed some projects using GO, then it is recommended to first remove all previous versions:

***sudo rm -rf /usr/local/go***

and then install the required version of GO.

Now the current versions are higher 1.17.0+

Let's start downloading, unpacking and installing GO:

***curl https://dl.google.com/go/go1.17.2.linux-amd64.tar.gz | sudo tar -C /usr/local -zxvf -***

# 3. Exporting variables

***cat <<'EOF' >>$HOME/.profile***

***export GOPATH=$HOME/go***

***export GO111MODULE=on***

***export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin***

***EOF***

***source $HOME/.profile***

4. We check the correctness of the installation:

***go version***

The output should contain:

go version go1.17.2 linux/amd64

### Install BitSong node
***get go-bitsong repository: git clone https://github.com/bitsongofficial/go-bitsong.git***

***git checkout v0.8.0***

***make install***

### INIT Node

***bitsongd init <_NAME MONIKER_> --chain-id bitsong-2b***

Download the correct genesis file::

***wget https://github.com/bitsongofficial/networks/raw/master/bitsong-2b/genesis.json -O ~/.bitsongd/config/genesis.json***

We edit the config.toml configuration file, add checked peers:

P2P Configuration Options 

nano config.toml

Вносим значения  поле:

***persistent_peers=***

a62038142844828483dbf16fa6dd159f6857c81b@173.212.247.98:26656,e9fea0509b1a2d16a10ef9fdea0a4e3edc7ca485@185.144.83.158:26656,8208adac8b09f3e2499dfaef24bb89a2d190a7a3@164.68.109.246:26656,cf031ac1cf44c9c311b5967712899391a434da9a@161.97.97.61:26656,d6b2ae82c38927fa7b7630346bd84772e632983a@157.90.95.104:15631,a5885669c1f7860bfe28071a7ec00cc45b2fcbc3@144.91.85.56:26656,325a5920a614e2375fea90f8a08d8b8d612fdd1e@137.74.18.30:26656,ae2787a337c3599b16410f3ac09d6918da2e5c37@46.101.238.149:26656,9336f75cd99ff6e5cdb6335e8d1a2c91b81d84b9@65.21.0.232:26656,9c6e52e78f112a55146b09110d1d1be47702df27@135.181.211.184:36656

***seeds*** = ffa27441ca78a5d41a36f6d505b67a145fd54d8a@95.217.156.228:26656,efd52c1e56b460b1f37d73c8d2bd5f860b41d2ba@65.21.62.83:26656

### Creating a wallet ###

***key bitsongd keys add YOUR-WALLET-NAME***

***MANDATORY we write mnemonics!!!!***

If the wallet has already been created, then it can be restored using this command and mnemonics:

***bitsongd keys add YOUR-WALLET-NAME --recover (it will ask your mnemonic)***

### INSTALL Cosmosvisor ###

To automate the installation of updated versions of binaries, it is recommended to install and use Cosmosvisor
  
Download and install Cosmovisor

### cd ~ ###

***git clone https://github.com/cosmos/cosmos-sdk***

***cd cosmos-sdk***

***git checkout v0.42.7***

***make cosmovisor***

***cp cosmovisor/cosmovisor $GOPATH/bin/cosmovisor***

***cd $HOME***

***Create the folders needed for the cosmovisor***

***mkdir -p ~/.bitsongd/cosmovisor***

***mkdir -p ~/.bitsongd/cosmovisor/genesis***

***mkdir -p ~/.bitsongd/cosmovisor/genesis/bin***

***mkdir -p ~/.bitsongd/cosmovisor/upgrades***

### Setting the required variables ###

***echo "# Setup Cosmovisor" >> ~/.profile***

***echo "export DAEMON_NAME=bitsongd" >> ~/.profile***

***echo "export DAEMON_HOME=$HOME/.bitsongd" >> ~/.profile***

***echo "export DAEMON_RESTART_AFTER_UPGRADE=true" >> ~/.profile***

***source ~/.profile***

### Copying the binary for launching the node into the cosmovisor ###

***cp $GOPATH/bin/bitsongd ~/.bitsongd/cosmovisor/genesis/bin/***

### Setting up a service file ###

sudo tee /etc/systemd/system/bitsongd.service > /dev/null <<EOF
                                                                
[Unit]
                                                                
Description=Nibiru Daemon
                                                                
After=network-online.target
                                                                
[Service]
                                                                
User=$USER
                                                                
ExecStart=$(which cosmovisor) start
                                                                
Restart=always
                                                                
RestartSec=3
                                                                
LimitNOFILE=65535
                                                                
Environment="DAEMON_HOME=$HOME/.bitsongd"
                                                                
Environment="DAEMON_NAME=bitsongd"
                                                                
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
                                                                
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
                                                                
[Install]
                                                                
WantedBy=multi-user.target
                                                                
EOF
                                                                
### Run the service file ###
                                                                
***sudo systemctl daemon-reload***
                                                                
***sudo systemctl enable bitsongd***
                                                                
***sudo systemctl start bitsongd***
    
                                                                
### Checking the status of the service ###
                                                                
***sudo systemctl status bitsongd***
                                                                
 ### Checking node logs ###

 ***journalctl -u bitsongd -f***
                                                                
### Checking node synchronization: ###
                                                                
***bitsongd status 2>&1 | jq***
  
or
  
***curl -s localhost:26657/status | jq .result.sync_info.catching_up***
  
If the result is issued 

***FALSE***
   
you can start creating a validator
  
  ### INSTALL VALIDATOR ###
  
 To complete the transaction to create an alidator, you need project tokens, so first you need to replenish your wallet with a certain amount of tokens!!!
  
 You can check the balance of the wallet using the command:
  
  ***bitsongd query bank balances <_wallet address_>***
  
  If there are available tokens on the balance, then we proceed to create a validator:
 
  bitsongd tx staking create-validator \
  
--amount=10000000ubtsg \
  
--pubkey=$(bitsongd tendermint show-validator) \
  
--moniker= ***your name moniker*** \
  
--chain-id=bitsong-2b \
  
--commission-rate="0.1" \
  
--commission-max-rate="1.00" \
  
--commission-max-change-rate="1.00" \
  
--min-self-delegation="1" \
  
--gas="auto" \
  
--gas-adjustment="1.2" \
  
--gas-prices="0.025ubtsg" \
  
--from= ***name key***
  
If the transaction is successful, go to the block explorer and look for your validator:
  
https://bitsong.bigdipper.live/validators


