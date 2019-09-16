Beginners Guide for Iquidus Explorer Setup

This Tutorial is going to create a Daemon (node) and install Explorer. Paisa coin is used here as example
THIS IS NOT GOING TO CREATE A GUI CLIENT.

Follow the instructions in [whatever coin name] docs folder Unix build - some builds are different.

I have tested and setup multiple time with help of this upon Ubuntu 16.04 with no issues.
You can create an account on Google Cloud and get $300 free to be used in 12 months or Create an account on Digital Ocean and 
get $100 free to be used in 12 months

IF YOU NEED HELP PLEASE CONTACT ME ON DISCORD (QJUSAM#1739) OR TELEGRAM (@SAMQJU)

Assuming you have created a fresh Ubuntu 16.04 LTS VPS follow these steps

*********************
Update System
*********************
sudo apt-get update
sudo apt-get upgrade -y

**********************************
Basic packages to build the coin
**********************************
sudo apt-get install build-essential libssl-dev libdb++-dev libboost-all-dev libqrencode-dev miniupnpc libminiupnpc-dev autoconf pkg-config libtool autotools-dev libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev automake -y

**********************************
Compile & Configure your coin 
**********************************
sudo git clone https://github.com/samqju/Paisa.git
cd Paisa/src/
sudo make -f makefile.unix

**********************************************************************************
or for any newer code/ different algo coin instructions may be different as below
**********************************************************************************
./autogen.sh
./configure
make
make install

**********************************************************************************
After compiling the coin daemon, Add RPC & Server settings by Create a conf file
**********************************************************************************
cd
sudo nano .paisa/paisa.conf

***************
Add these lines
***************
rpcuser=paisacoin4u
rpcpassword=mylove4u
listen=1
daemon=1
server=1
txindex=1
rpcport=XXXXXX (here use your coin's actual rpc port)

**********************
Start the coin daemon
**********************
cd Paisa/src
./paisad -daemon -txindex

****************
Install MongoDB
****************
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6

echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list

sudo apt-get update
sudo apt-get install -y mongodb-org

************************************************************************
check if mongodb is installed and working correctly, with these commands
************************************************************************
sudo service mongod start
cd /var/log/mongodb
tail mongod.log

*************************************
Last line of output will be like this 
*************************************
# [initandlisten] waiting for connections on port 27017

**************
Setup MongoDB
**************
mongo
use explorerdb
db.createUser( { user: "iquidus", pwd: "3xp!0reR", roles: [ "readWrite" ] } )
exit

***********************
Install Node.js and Npm
***********************
sudo apt-get update
sudo apt-get install nodejs nodejs-legacy -y
sudo apt-get install npm -y

*******************************
Install Iquidis Block Explorer
*******************************
cd
git clone https://github.com/iquidus/explorer.git
sudo apt-get install libkrb5-dev -y
cd explorer && npm install --production
cp ./settings.json.template ./settings.json

*****************
Configure Iquidis - IF YOU NEED HELP PLEASE CONTACT ME ON DISCORD OR TELEGRAM
*****************
nano settings.json

"title": "EXPLORER TITLE",
"address": "explorer link",  
"coin": "NAME OF COIN",
"symbol": "coin ticker",

 // wallet settings
  "wallet": {
    "host": "localhost",
    "port": "your coin rpc port,
    "user": "paisacoin4u",
    "pass": "mylove4u"
    
 // menu settings
  "display": {
    "markets": true, - Only supported if your coin is listed in bittrex or cryptopia

 // index page (valid options for difficulty are POW, POS or Hybrid)
  "index": {
    "difficulty": "POW", (change it to the option suitable for your coin)

  // ensure links on API page are valid
  "api": {
    "blockindex": 1, - basically any valid block number of your coin's blockchain
    "blockhash": "xxxxxxxxxxxxxxxxx", - from your local wallet get block hash of any any valid block of your coin's blockchain, 
    "txhash": "xxxxxxxxxxxxxxxxx", - from your local wallet get txid of any any valid block of your coin's block chain,
    "address": "xxxxxxxxxxxxxxxxxxxxxx" any valid address on your coin's network
  },
  
// market settings
     "coin": "xxx", add your coin's ticker
     
//genesis
  "genesis_tx": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx", update with your coin's genesis block details, "getblock 0 or showblock 0"
  "genesis_block": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",

******************************
Start Explorer & Sync Database
******************************
cd /root/explorer
screen -S npm - This will start a screen service which will make it easy for you to monitor things
npm start - this will start explorer service, now you can exi the screen by pressing Ctrl+A+D

cd /root/explorer
screen -S update - This will start a screen service which will make it easy for you to monitor things
sudo node scripts/sync.js index update - this will start sync and now you will start seeing block numbers getting synced

EXIT THE SCREEN AGAIN BY PRESSING CTRL+A+D

NOW GO TO YOUR EXPLORER LINK AND YOU CAN SEE BLOCKS GETTING UPDATED, ONCE SYNC IS COMEPLETED YOU NEED TO CREATE CRONTABS SO THAT YOUR EXPLORER GETS SYNC REGULARLY

***************
Adding Crontabs
***************
sudo crontab -e - here it might ask you which editor you want to use I prefer nano you can choose anyone

*************************************
Add these lines at the bottom of file
*************************************
*/1 * * * * cd /root/explorer && /usr/bin/nodejs scripts/sync.js index update > /dev/null 2>&1  
*/2 * * * * cd /root/explorer && /usr/bin/nodejs scripts/sync.js market > /dev/null 2>&1  
*/5 * * * * cd /root/explorer && /usr/bin/nodejs scripts/peers.js > /dev/null 2>&1 
