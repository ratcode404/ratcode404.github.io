sudo apt upgrade 
sudo apt update

sudo apt install build-essential bison gperf cmake libsqlite3-dev libaspell-dev libpcre3-dev nettle-dev g++ libcurl4-openssl-dev libargon2-dev libssl-dev

git clone https://github.com/lisdude/toaststunt
git clone https://github.com/lisdude/toastcore
cp toastcore/toastcore.db toaststunt/toastcore.db

cd toaststunt
sudo mkdir build && cd build
sudo cmake ../
sudo make -j2

 ./moo ../toastcore.db <database-name>.db

Control Z
jobs
bg 1

telnet localhost 7777

connect wizard

Set a password for yourself.
  -- @password <new-password>

@set $network.MOO_Name to "<game name>"
https://www2.hawaii.edu/~herve/MOO/


git clone https://github.com/JavaChilly/dome-client.js
sudo apt install npm
sudo apt upgrade && sudo apt update
sudo npm install
sudo npm init
sudo npm install -g forever
nano dome-client.js/config/default.js
