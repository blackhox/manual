Download files
wget https://storage.googleapis.com/pryzm-zone/feeder/config.yaml
get https://storage.googleapis.com/pryzm-zone/feeder/pryzm-feeder.tgz
Unpack:
tar -xvzf pryzm-feeder.tgz
Installing Node.js and pnpm
curl -sL https://deb.nodesource.com/setup_20.x -o nodesource_setup.sh
sudo apt install nodejs
Checking the installation and version
node -v
npm -v
Updating to the latest version
npm install -g npm@10.2.5
npm install -g pnpm
pnpm -v

Install postgresql,setting a system password, creating a database

sudo apt install postgresql
sudo systemctl start postgresql.service
sudo systemctl status postgresql.service

sudo -u postgres psql
ALTER USER postgres WITH PASSWORD 'postgres';
CREATE DATABASE "pryzm-feeder";

View a list of all databases:
sudo -u postgres psql
\l
Exit from Postgres console:
\q
Creating a feeder key, or restoring it if you decide to use a validator key
pryzmd keys add Name_key
pryzmd keys add Name_key --recover

Go to the feeder directory

cd $HOME/feeder/
mkdir app
We copy the previously downloaded config.yaml file into the created app directory and enter all the necessary data into it
After entering the data into the configuration file, run the feeder script vote.js
node ./lib/vote.js $HOME/feeder/app/config.yaml

We look at the script logs, if everything went without errors, then we stop the execution of the script and run it either in Screen or in Tmux,
or create an autorun daemon using third-party packages forever or PM2.
