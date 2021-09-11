# This manual contains information that will allow you to successfully create relay channels and relay between the kichain-t-4 and Cygnusx-Osmo-1 testnets.

To complete the task, you need to install a client of the desired network on your server and create a wallet there, which will contain test tokens.

The instruction will be divided into two thematic parts: installing the network client and installing and configuring the relay itself.

If you plan to install a node on a clean Linux operating system, then you need to perform the initial basic configuration

The first step describes the basic requirements and settings:

# Operating system
This challenge was completed on a cloud instance running ubuntu 20.04 LTS.

# Install toolchain

$ sudo apt update && sudo apt install make clang pkg-config libssl-dev build-essential git jq llvm libudev-dev -y

# Install GO

If the OS on the server is not “clean” some projects were previously installed, then it is recommended to first delete all previous versions:

sudo rm -rf /usr/local/go

Let’s start downloading, unpacking and installing GO:

curl https://dl.google.com/go/go1.16.7.linux-amd64.tar.gz | sudo tar -C /usr/local -zxvf -

***We export the variables:***

cat <<’EOF’ >>$HOME/.profile

export GOPATH=$HOME/go

export GO111MODULE=on

export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin

EOF

source $HOME/.profile

***We check the correctness of the installation, run the command:***

go version

The output should contain:

go version go1.16.7 linux / amd64

For the first part, I chose the Cygnusx-Osmo-1 network — the test network was launched by the Stargaze team. I chose this network because I participated in the testnet of this project and I have a fully configured node with a wallet and tokens.
The installation of the network client will be carried out on a working node of the KiChain network with the created wallet address, which already has network tokens.


# PART I: INSTALLING THE NETWORK CLIENT

Let’s start by installing the network client Osmosis:

1.	Download and install the Osmosis network client

git clone https://github.com/osmosis-labs/osmosis

 cd osmosis
 
 git checkout v1.0.1
 
 make install
 
 cd ~
 
 
If you participated in the Stargaze testnet and you already have a wallet with a balance in this network, you can restore it using the seed phrase:

***osmosisd keys add WALLET_NAME_OSMO –recover***

there will be a request to enter mnemonic phrases that needed to be recorded during the initial creation of the wallet

If you have never had a wallet on this network, then you can create it using the following command:

***osmosisd keys add WALLET_NAME_OSMO***

(we save all the output lines, there will be a seed phrase from the wallet, which must be saved)

***Keys for testnets OSMO:***

osmo1v4fx46k46zgy42qtf6mnmsw4a23aujhuhwkgrm

You can view your wallet address using the following command:

***osmosisd keys show OSMO_WALLET_NAME -a***

To the created or restored wallet address, you need to request testnet tokens, in our case Cygnusx-Osmo-1.

You can request them either from the network validators or from the faucet.

Since I took part in this testnet, I have tokens on the restored wallet.

After a request with a token, you need to receive confirmation that they have come to your wallet address. This can be done using the command:

***osmosisd query bank balances OSMO_WALLET_ADDRESS --node http://54.166.148.90:36657***

If the tokens are on the balance of your wallet, then you can read the first part of this guide and you can proceed to the second part.

# PART II. RELAY START

1.	Download and install the relay:

git clone https://github.com/cosmos/relayer.git

 cd relayer
 
 make install
 
 cd
 
2. We initialize the relay using the following command:

***rly config init***

3.Create a folder in which we will put folders with data for the configuration of the relay and go to it

***mkdir rly_config***

 ***cd rly_config***
 
4. We create configs for the networks used:

This can be done using the nano text editor, or any other:

Config for the Ki network:

***nano kichain-t-4_config.json***

Inside the file, insert such data

{

 “chain-id”: “kichain-t-4”,
 
 “rpc-addr”: “http://127.0.0.1:26657",
 
 “account-prefix”: “tki”,
 
 “gas-adjustment”: 1.5,
 
 “gas-prices”: “0.025utki”,
 
 “trusting-period”: “48h”
 
 }
 
Config for the Osmosis:

***nano cygnusx-osmo-1_config.json***

***Inside the file, insert such data***

{

 “chain-id”: “cygnusx-osmo-1”,
 
 “rpc-addr”: “http://54.166.148.90:26657",
 
 “account-prefix”: “osmo”,
 
 “gas-adjustment”: 1.5,
 
 “gas-prices”: “0.025uosmox”,
 
 “trusting-period”: “48h”
 
 }
 
5. Add the pre-written settings to the config file of the relay and go to the root directory

***rly chains add -f kichain-t-4_config.json***

***rly chains add -f cygnusx-osmo-1_config.json***

 ***cd***
 
6. Let’s create and add separate wallets to the relayer.

Do not forget that it is imperative to save the output of these commands, because they contain seed phrases

***rly keys add kichain-t-4 kidrly***

***rly keys add cygnusx-osmo-1 osmorly***

In this task, I created two relayer wallets on two networks:

***kichain-t-4:***

tki12r8vskw5sjt8c64zuexnt6u9ntlzvcw30xkvs3

***cygnusx-osmo-1***

osmo1zk4dcdzk8yrt6kcr8aamdq94n5999600du9svz

7. Add the created wallets to the config of the relay

***rly chains edit kichain-t-4 key kidrly***

***rly chains edit cygnusx-osmo-1 key osmorly***

8. Change the waiting timeout in the relay settings so that it can accurately process transactions

***nano ~/.relayer/config/config.yaml***

Find the line:

***timeout: 10s***

and change the timeout value in it:

***timeout: 3m***

9. We replenish the purses of the relay for any amount, you can use 5–10 coins. In this example, I will send 10 coins

***kid tx bank send WALLET_NAME_KICHEIN_WALLET_ADDRESS_kidrly 10000000utki --home PATH_TO_YOUR_NODE_DIRECTORY --chain-id kichain-t-4***

***osmosisd tx bank send WALLET_NAME_OSMO WALLET_ADDRESS_osmorly 10000000uosmox --node http://54.166.148.90:26657/ --chain-id cygnusx-osmo-1 --fees 5000uosmox***

10. Check balances of relayers’ wallets:

***rly q balance kichain-t-4***

***rly q balance cygnusx-osmo-1***

11. After the tokens arrive on the wallets of the relay, we initialize clients for both networks

***rly light init kichain-t-4 -f***

***rly light init cygnusx-osmo-1 -f***

12. Create a tunnel between two networks

***rly paths generate kichain-t-4 cygnusx-osmo-1 transfer — port=transfer***

13. Check if it was successfully created

***rly paths list -d***

If the tunnel was created successfully, the following output will be displayed on the screen:

0: transfer -> chns (✔) clnts (✔) conn (✔) chan (✔) (kichain-t-4: transfer <> cygnusx-osmo-1: transfer)

In this case, you can skip the next step and start creating a service file.

If the channel could not be created immediately, then the output of the previous command will look like this:

0: transfer -> chns(✔) clnts(x) conn(x) chan(x) (kichain-t-4:transfer<>cygnusx-osmo-1:transfer)

then go to the next step:

We create clients, channels and connections in the connected networks, which are necessary for the operation of the tunnel.

For this we execute the command:

***rly tx link transfer***

We are waiting for the team to finish the work. The last line will contain the following fragment

★ Channel created:

I want to draw your attention to the fact that the execution time of this command can take a very long time. Even a few hours.
It is imperative to wait until the end and not interrupt its execution

If your transaction did not go through at some stage or ended with an error, then you need to open the configuration file of the relay

***nano ~ / .relayer / config / config.yaml***

and remove the lines in the paths section:

***client-id: …***

***connection-id: …***

***channel-id: …***

in ***src*** and ***dst***

then re-conduct the transaction.

14. After successful completion of the transaction, we re-check if all the checkboxes appear in the output of the rly paths list -d command. If they appear, create a service file.
(ATTENTION: this guide assumes that a file called kichaind.service is used to start the node. 
If it is different for you, replace it with the correct one in the [Unit] section of the service file):

sudo tee /etc/systemd/system/rlyd.service > /dev/null <<EOF
                                                            
 [Unit]
                                                            
 Description=relayer client
                                                            
 After=network-online.target, kichaind.service[Service]
                                                            
 User=$USER
                                                            
 ExecStart=$(which rly) start transfer
                                                            
 Restart=always
                                                            
 RestartSec=3
                                                            
 LimitNOFILE=65535[Install]
                                                            
 WantedBy=multi-user.target
                                                            
 EOF
                                                            
15. Start the service file
                                                            
***sudo systemctl daemon-reload***
                                                            
***sudo systemctl enable rlyd***
                                                            
***sudo systemctl start rlyd***
                                                            
To check the logs of the relay, you need to use the following command:
                                                            
***journalctl -u rlyd –f***
                                                            
16. To carry out an ibc transaction, you can use both the subcommands of the relay and, accordingly, the wallets that are added to it, as well as the subcommands of the clients of the
networks and the wallets added to them.
                                                            
For example:
                                                            
Relayer:
                                                            
***rly tx transfer kichain-t-4 cygnusx-osmo-1 1000000utki USER_ADDRESS_On_OSMO_NETWORK --path transfer***
                                                            
***Client Kichen:***
                                                            
***kid tx ibc-transfer transfer transfer CHANNEL_ID RECIPIENT_ADDRESS IN_OSMO_NET 1000000utki --from WALLET_NAME_KICHEIN --home PATH_TO_YOUR_NODE_DIRECTORY_chain-id kichain-t-4***
                                                            
To get the CHANNEL_ID you need to execute the command:
                                                            
***rly paths show transfer --yaml***
                                                            
you need the value of the channel-id item of the src section
                                                            
17. After the transaction, we check the balance of the recipient’s wallet to make sure that the tokens have arrived
                                                            
***osmosisd q bank balances RECEIVER_ADDRESS_IN_OSMO_NETWORK --node http://54.166.148.90:26657/***
                                                            
If the transaction is successful, then the output of the balance request command will look like this:
                                                            
balances:
                                                            
- amount: “1000000”
                                                            
denom: ibc/098EEBA8FD704F5CB8444C6FA9180257D333CB42F43626A22AC754B304B8EAF8
                                                            
- amount: “0”
                                                            
denom: ibc/F231E772EC3A8A475959441AA087BE99F1DC11FDA13ABF07BC697A9B85EED2DD
                                                            
The results of the task can be seen in the explorer of both the Kichain network and the Cygnusx-Osmo-1 network:
                                                            
***Kichain network:***
                                                            
https://ki.thecodes.dev/tx/CE7BFEEA4A26CD4C800459DF3E895780DEC586C9870B5E4F0173688DAD76BA9F
                                                            
https://ki.thecodes.dev/tx/C8D8DB7758DA827673E24FD03D4EC68BB3A4F6512430858152DFFC56C282D3B3
                                                            
https://ki.thecodes.dev/tx/1F93C9926D3F883480DC527824A25996C4B73E4B60D12C6CB8A70DE434A147BB
                                                            
https://ki.thecodes.dev/tx/AEF91B09DEA0DE799CA2A647CB9CD3E2849F0A6652D6BB757AA716CE0FAB6FE9
                                                            
https://ki.thecodes.dev/tx/8956DC93AB895E995F8E821FAC38DDEDC2B97B05843908D4B811C812FE0D9AAA
                                                            
https://ki.thecodes.dev/tx/46FBDF43C4D441CCBFD8C0B8784FD93FDDEE07BD3CCC1EFFE04980414D8C1EDC
                                                            
***Cygnusx-Osmo-1 network:***
                                                            
https://osmosis-explorer.cygnusx-1.publicawesome.dev/transactions/24C03C798D77098132ABA068E70E61FE304ABDC6714E894D67B413C1685C5FD6
                                                            
https://osmosis-explorer.cygnusx-1.publicawesome.dev/transactions/AF5CAFE6B550ED9FD5F0502DD7280D6CB4516B6BC030D320149CDC93FFDC7ECD
                                                            
https://osmosis-explorer.cygnusx-1.publicawesome.dev/transactions/3DB9C74CFEF1E83BED7BF27B2A0CA280F41EB4404D9DB139A44C32F3B7362533
                                                            
https://osmosis-explorer.cygnusx-1.publicawesome.dev/transactions/ED9856586FDE3480EAF53FB71AD13C96D99794C7939AE66BB6124A2925FF7171
                                                            
https://osmosis-explorer.cygnusx-1.publicawesome.dev/transactions/157598404A12541F75B4585A1C8EB71890D7F3BFF8A1A7C077EAABD5DFC9F184
                                                            
https://osmosis-explorer.cygnusx-1.publicawesome.dev/transactions/B8CE6F537D253F0240A29D34E3EC48FE4356640FC4ECD838BDA4A8151A3AC6EB
                                            
