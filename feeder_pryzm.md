# 1. Download files
wget https://storage.googleapis.com/pryzm-zone/feeder/config.yaml

wget https://storage.googleapis.com/pryzm-zone/feeder/pryzm-feeder.tgz

**Unpack:**
tar -xvzf pryzm-feeder.tgz
# 2. Installing Node.js and pnpm
```curl -sL https://deb.nodesource.com/setup_20.x -o nodesource_setup.sh```

```sudo apt install nodejs```

***Checking the installation and version***

```node -v```

```npm -v```

***Updating to the latest version***

```npm install -g npm@10.2.5```

```npm install -g pnpm```

```pnpm -v```

# 3. Install postgresql,setting a system password, creating a database

```sudo apt install postgresql```

```sudo systemctl start postgresql.service```

```sudo systemctl status postgresql.service```

```sudo -u postgres psql```

```ALTER USER postgres WITH PASSWORD 'postgres';```

```CREATE DATABASE "pryzm-feeder";```

***View a list of all databases:***

```sudo -u postgres psql```

```\l```

***Exit from Postgres console:***

```\q```

# 4. Creating a feeder key, or restoring it if you decide to use a validator key

```pryzmd keys add Name_key```

```pryzmd keys add Name_key --recover```

***Deposit tokens to the balance of the feeder wallet***

https://testnet.pryzm.zone/faucet

***We make changes to the app.toml configuration file***

```minimum-gas-prices = "0.015upryzm,0.01factory/pryzm15k9s9p0ar0cx27nayrgk6vmhyec3lj7vkry7rx/uusdsim"```

***Go to the feeder directory***

```cd $HOME/feeder/```

```mkdir app```

We copy the previously downloaded config.yaml file into the created app directory and enter all the necessary data into it
After entering the data into the configuration file, run the feeder script vote.js

```node ./lib/vote.js $HOME/feeder/app/config.yaml```

We look at the script logs, if everything went without errors, then we stop the execution of the script and run it either in Screen or in Tmux,
or create an autorun daemon using third-party packages forever or PM2.
# 5. Associate the feeder key with the validator
```pryzmd tx oracle delegate-feed-consent [feeder-address] --from {address wallet validator} --fees 2000factory/pryzm15k9s9p0ar0cx27nayrgk6vmhyec3lj7vkry7rx/uusdsim,3000upryzm```
