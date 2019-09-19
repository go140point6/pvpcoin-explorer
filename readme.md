# Iquidus Blockchain Explorer and PvPCoin

Ubuntu 18.04.03 LTS

### Update system and get dependencies
```
sudo apt-get update
sudo apt-get dist-upgrade -y
sudo apt-get install vim curl git screen net-tools python build-essential
```

### PvPCoin wallet
Note: Build if you want, but static working wallet is (https://github.com/go140point6/pvpcoin-build/releases)

Note: Special characters are a pain with using Iquidus, don't use them in the rpcpassword

playervsplayer.conf:
```
server=1
txindex=1
daemon=1
listen=1
maxconnections=512
rpcuser=pvpexplorer
rpcpassword={longandrandomRPCpassword}
rpcallowip=127.0.0.1
rpcconnect=127.0.0.1
rpcport=4567
addnode=140.186.30.102:4568
addnode=176.254.153.206:4568
addnode=162.237.70.56:56816
addnode=162.237.70.56:58343
addnode=162.237.70.56:59114
addnode=136.144.171.201:4568
addnode=22.137.226.5:4568
addnode=136.144.171.201:4568
addnode=155.138.162.120:4568
addnode=207.246.108.36:4568
```

### NVM (Node Version Manager)
Cleaner way to install new Node versions in independent directories (great for testing new version without messing with a working version)

Note: as of this writing, version 0.34.0 is the latest, always check and use the latest
```
mkdir nvm && cd nvm
curl -sL https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh -o install_nvm.sh && chmod +x install_nvm.sh
./install_nvm.sh
source ~/.bashrc

nvm ls-remote
```
 (this shows you all versions, look for the latest Node LTS version - currently 10.16.3)
```
nvm install 10.16.3
```

### MongoDB for Ubuntu 18.04
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 68818C72E52529D4
sudo echo "deb http://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org

sudo systemctl enable mongod
sudo systemctl start mongod
```

##### Secure MongoDB
```
mongo
use admin
db.createUser( { user: "admin", pwd: "longandrandomADMINDBpassword", roles: [ {role:"root", db: "admin"} ] } )
exit

edit -> /lib/systemd/system/mongod.service
line 9, add --auth:
ExecStart=/usr/bin/mongod --auth --config /etc/mongod.conf
edit -> /etc/mongod.conf
change default port to something other than standard, i.e. 19734
sudo systemctl daemon-reload
sudo systemctl restart mongod.service

mongo --port=19734 -u admin -p
```
Note: this prompts for mongo admin password so it's not saved in shell history
```
use pvpexplorerdb
db.createUser( { user: "pvpcoin", pwd: "longandrandomPVPEXPLORERDBpassword", roles: [ "readWrite" ] } )
exit
```

### Install and configure iquidus
```
sudo apt-get install libkrb5-dev -y
sudo chown -R $USER:$(id -gn $USER) ~/.config
mkdir ~/nvm && cd ~/nvm
git clone https://github.com/iquidus/explorer.git
cd explorer && npm install --production
```
Note: we will go back and fix vulnerabilities as best we can after iquidus is installed and tested working
```
cp ./settings.json.template ./settings.json
pvp.png -> copy to public/images/
cd public/images && chmod -x pvp.png
favicon.ico -> cp favicon.ico to public/
```

#### settings.json
```
/*
  This file must be valid JSON. But comments are allowed
  Please edit settings.json, not settings.json.template
*/
{
  // name your instance!
  "title": "PvPCoin",

  "address": "127.0.0.1:3001",

  // coin name
  "coin": "PvPCoin",

  // coin symbol
  "symbol": "PVP",

  // logo
  "logo": "/images/pvp.png",

  // favicon
  "favicon": "public/favicon.ico",

  // Uses bootswatch themes (http://bootswatch.com/)
  // Valid options:
  //     Cerulean, Cosmo, Cyborg, Darkly, Flatly, Journal, Lumen, Paper,
  //     Readable, Sandstone, Simplex, Slate, Spacelab, Superhero, United, Yeti
  // theme (see /public/themes for available themes)
  "theme": "Darkly",

  // port to listen for requests on.
  "port" : 3001,

  // database settings (MongoDB)
  "dbsettings": {
    "user": "pvpcoin",
    "password": "longandrandomPVPEXPLORERDBpassword",
    "database": "pvpexplorerdb",
    "address": "127.0.0.1",
    "port": 19734
  },

  //update script settings
  "update_timeout": 10,
  "check_timeout": 250,

  // wallet settings
  "wallet": {
    "host": "127.0.0.1",
    "port": 4567,
    "user": "pvpexplorer",
    "pass": "longandrandomRPCpassword"
  },

  // confirmations
  "confirmations": 12,

  // language settings
  "locale": "locale/en.json",

  // menu settings
  "display": {
    "api": true,
    "markets": false,
    "richlist": true,
    "twitter": true,
    "facebook": false,
    "googleplus": false,
    "youtube": false,
    "search": true,
    "movement": true,
    "network": true
  },

  // index page (valid options for difficulty are POW, POS or Hybrid)
  "index": {
    "show_hashrate": true,
    "difficulty": "POW",
    "last_txs": 100
  },

  // ensure links on API page are valid
  "api": {
    "blockindex": 6413,
    "blockhash": "0000000000022471a545fc3bd36d2d38a9dfab9d701a49789f2fae88d4eb5e3f",
    "txhash": "26853075f134707631200e1994bbd7379ea659cf05b5f04e21c50a5d06065a11",
    "address": "PLvaDAEK6BsG7PmnYPbHPiyyzKqivBWDAU"
},

  // market settings
  //supported markets: bittrex, poloniex, yobit, empoex, bleutrade, cryptopia, ccex
  //default market is loaded by default and determines last price in header
  "markets": {
    "coin": "PVP",
    "exchange": "BTC",
    "enabled": ["bittrex"],
    "cryptopia_id": "1658",
    "ccex_key" : "Get-Your-Own-Key",
    "default": "bittrex"
  },

  // richlist/top100 settings
  "richlist": {
    "distribution": true,
    "received": true,
    "balance": true
  },
  // movement page settings
  // min amount: show transactions greater than this value
  // low flag: greater than this value flagged yellow
  // high flag: greater than this value flagged red
  "movement": {
    "min_amount": 22,
    "low_flag": 100,
    "high_flag": 1000
  },

  // twitter, facebook, googleplus, youtube
  "twitter": "PureNoel",
  "facebook": "yourfacebookpage",
  "googleplus": "yourgooglepluspage",
  "youtube": "youryoutubechannel",

  //genesis
  "genesis_tx": "1c1d09740e69c1b201ccfe83be3a7b7692adca82c12265f8c5db3cfdd665f05d",
  "genesis_block": "0000052a0dc9a67b2f7752fc819b863114fc65d984dfd8e704f698bea694cfe5",

  //heavy (enable/disable additional heavy features)
  "heavy": false,

  //amount of txs to index per address (stores latest n txs)
  "txcount": 100,

  //show total sent & received on address page (set false if PoS)
  "show_sent_received": true,

  // how to calculate current coin supply
  // COINBASE : total sent from coinbase (PoW)
  // GETINFO : retreive from getinfo api call (PoS)
  // HEAVY: retreive from heavys getsupply api call
  // BALANCES : total of all address balances
  // TXOUTSET : retreive from gettxoutsetinfo api call
  "supply": "COINBASE",

  // how to acquire network hashrate
  // getnetworkhashps: uses getnetworkhashps api call, returns in GH/s
  // netmhashps: uses getmininginfo.netmhashpsm returns in MH/s
  "nethash": "getnetworkhashps",

  // nethash unitd: sets nethash API return units
  // valid options: "P" (PH/s), "T" (TH/s), "G" (GH/s), "M" (MH/s), "K" (KH/s)
  "nethash_units": "G",

  // Address labels
  // example : "JhbrvAmM7kNpwA6wD5KoAsbtikLWWMNPcM": {"label": "This is a burn address", "type":"danger", "url":"http://example.com"}
  // label (required) = test to display
  // type (optional) = class of label, valid types: default, primary, warning, danger, success
  // url (optional) = url to link to for more information
  "labels": {
  //  "JSoEdU717hvz8KQVq2HfcqV9A79Wihzusu": {"label": "Developers address", "type":"primary", "url":"http://example.com"},
  //  "JSWVXHWeYNknPdG9uDrcBoZHztKMFCsndw": {"label": "Cryptsy"}
  }
}
```

#### Fix for 'length' undefined
edit -> lib/explorer.js
```
- 343 if (tx) {
+ 343 if (tx && tx.vout) {
```

edit -> scripts/sync.js
```
+ 107 // unlink lock file
+ 108 process.on('uncaughtException', exit);
```

### Start up and get initial index
```
cd ~/nvm/explorer
```
Note: I find it easiest to open multiple ssh shells here, but you should use screen no matter what to keep the explorer running. The information shown here is very helpful to troubleshoot.  Be sure to exit the 'npm' screen with ctrl+A+D to leave it running in the background.
```
screen -S npm
npm start
ctrl+A+D

screen -S update
node scripts/sync.js index reindex
node scripts/sync.js index update
```
If all goes well, the 'index update' should start at the genesis block and index all blocks (output to screen), takes a little while to complete ubt you'll definitely know it's working.  When done, you can exit the 'update' screen

### nginx reverse proxy
```
sudo apt-get update
sudo apt-get install nginx
sudo systemctl enable nginx.service

sudo unlink /etc/nginx/sites-enabled/default
edit -> /etc/nginx/sites-available/pvpexplorer
add:
  server {
      listen 80;
      server {what your site/domain is}
  
      location / {
          proxy_set_header   X-Forwarded-For $remote_addr;
          proxy_set_header   Host $http_host;
          proxy_pass         "http://127.0.0.1:3001";
      }
}
sudo ln -s /etc/nginx/sites-available/pvpexplorer /etc/nginx/sites-enabled/pvpexplorer
sudo systemctl restart nginx
```

If all goes well, you should be able to access your IP address at port 80 and see the PvP Explorer.  You should see current info if the script above ran correctly.

### go back and fix npm vulnerabilities that you can
Note: you won't be able to fix everything, look for the upcoming release of iquidus to help address some of what is left
```
cd ~/nvm/explorer
screen -r npm
ctrl-x
npm audit && npm audit fix
npm install && npm audit fix

npm install mongoose@5.7.1 && npm audit fix
npm install --save-dev jasmine@3.4.0 && npm audit fix
npm install debug@4.1.1 && npm audit fix

npm start
ctrl+A+D
```

### cron for running the update script every 2 minutes
```
crontab -e

*/2 * * * * cd /home/{username}/nvm/explorer/ && /home/{username}/.nvm/versions/node/v10.16.3/bin/node scripts/sync.js index update > /dev/null 2>&1
```

Might be more that you can do to secure and improve the setup.  There is something called 'forever' that you use to keep node.js running, but I didn't really play with it.  So far it's stayed running for me, but may be worth exploring.  If so, please let me know.
