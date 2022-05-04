## BTCIX Mainnet client

Golang implementation for BTCIX Network

## Mainnet information [For geth, also mentioned in config.toml]
```shell
NetworkId = 19845
SyncMode = "full"
```
## Public Chain information [For metamask]
```shell
Chain ID: 19845
Mainnet RPC URL : seed.btcix.org
Block Explorer: http://btcixscan.com
Symbol:BTCIX
```


### Programmatically interfacing `geth` nodes

HTTP based JSON-RPC API options:

  * `--http` Enable the HTTP-RPC server
  * `--http.addr` HTTP-RPC server listening interface (default: `localhost`)
  * `--http.port` HTTP-RPC server listening port (default: `8545`)
  * `--http.api` API's offered over the HTTP-RPC interface (default: `eth,net,web3`)
  * `--http.corsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
  * `--ws` Enable the WS-RPC server
  * `--ws.addr` WS-RPC server listening interface (default: `localhost`)
  * `--ws.port` WS-RPC server listening port (default: `8546`)
  * `--ws.api` API's offered over the WS-RPC interface (default: `eth,net,web3`)
  * `--ws.origins` Origins from which to accept websockets requests
  * `--ipcdisable` Disable the IPC-RPC server
  * `--ipcapi` API's offered over the IPC-RPC interface (default: `admin,debug,eth,miner,net,personal,shh,txpool,web3`)
  * `--ipcpath` Filename for IPC socket/pipe within the datadir (explicit paths escape it)
If you want to access RPC from other containers
and/or hosts then you can expose your http rpc by `--http.addr 0.0.0.0` [ Caution: this will expose it to all of btcix geth nodes]. By default, `geth` binds to the local interface and RPC endpoints is not
accessible from the outside. there are secure ways to do it using nginx security which adds http authentication over geth layer.
Please follow extra_security.md for nginx security.

### Operating a geth full node
# Become Root
```
sudo su
```

# Create bitco User
```
sudo useradd -m bitco
```

# Switch To The New User's Home
```
cd /home/bitco
```

# Download The Client
```
wget https://github.com/Bitcolojix/btcix/releases/download/v1.10.14/geth_linux
chmod +x geth_linux
wget https://github.com/Bitcolojix/btcix/releases/download/v1.10.14/btcix_genesis.json
wget https://github.com/Bitcolojix/btcix/releases/download/v1.10.14/config.toml
./geth_linux init btcix_genesis.json -datadir ./btcix 
```
Above command will end with following message "Successfully wrote genesis state "
it will also create a directory called btcix in /home/bitco

# Create `start.sh` and `console.sh`
```

echo './geth_linux --http --http.addr '127.0.0.1' --http.corsdomain "*" --http.port 8503 --http.api 'eth,net,web3,txpool,personal'  --allow-insecure-unlock --config config.toml' > start.sh

chmod +x start.sh

echo "./geth_linux attach ipc:btcix/geth.ipc" > console.sh
chmod +x console.sh

[Optional]

echo "./geth_linux --datadir ./btcix "$@"" > cli.sh
chmod +x cli.sh
```

# Setup systemd
```
sudo nano /lib/systemd/system/btcix.service
```

Then paste the following;

```
[Unit]
Description=Btcix Full node

[Service]
User=bitco
Type=simple
WorkingDirectory=/home/bitco
ExecStart=/bin/bash /home/bitco/start.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

After that;

```
chown -R bitco.bitco /home/bitco/*
systemctl enable btcix
systemctl start btcix
```


# Check info of node
```
./console.sh
```
# CLI Usages

```

./cli.sh account new
./cli.sh account list

```
# Securing Node
```
Create alpha numeric strings from here
1. 6+ chars long  nginx username
2. 32 chars long  nginx password


nginx user:MMxf9RMA
nginx Pass:DkfjW3FJSGLlBg0XvYrMudFDTCFszmpYYvkc


```
# Creating First account
```
./cli.sh account new

Or 

./console.sh

then type 

 personal.newAccount()
passPhrase:
Repeast Password:
prompt will ask
Your new account is locked with a password. Please give a password. Do not forget this password.
Enter your first account pass
Password:claEHfMZmHvY5dqoTU6BtHIecFhV4Z6SFJjt
Repeat Password:claEHfMZmHvY5dqoTU6BtHIecFhV4Z6SFJjt

Result
Public address of the key:   0xG124634534650A355456453665436544365
Path of the secret key file: mainnet/keystore/UTC--2022-01-06T06-46-38.255700718Z--G124634534650A355456453665436544365

- You can share your public address with anyone. Others need it to interact with you.
- You must NEVER share the secret key with anyone! The key controls access to your funds!
- You must BACKUP your key file! Without the key, it's impossible to access account funds!
- You must REMEMBER your password! Without the password, it's impossible to decrypt the key!
```
Cross Checking if password you entered is correct [its must to avoid loosing funds]

> personal.unlockAccount("0xG124634534650A355456453665436544365")
Unlock account claEHfMZmHvY5dqoTU6BtHIecFhV4Z6SFJjt
Passphrase:claEHfMZmHvY5dqoTU6BtHIecFhV4Z6SFJjt
true

When it says true then it means password is correct .

@todo 
Now backup following file [This is a json file ]

mainnet/keystore/UTC--2021-08-27T06-46-38.255700718Z--G124634534650A355456453665436544365

and save your password for first account somewhere same.

# Nginx Security Installation

Installing nginx on Ubuntu 20 ref: https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04
```
sudo apt update
sudo apt install nginx
systemctl status nginx
```


@todo adjust firewall to allow ip of nginx from only your reliable ip
```
sudo sh -c "echo -n 'MMxf9RMA:' >> /etc/nginx/.htpasswd"
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"
```
password for nginx=DkfjW3FJSGLlBg0XvYrMudFDTCFszmpYYvkc

```
sudo nano /etc/nginx/sites-available/default
```
``` Sample NGINX Config
server {
 listen 8503;
 listen [::]:8503;
 # ADDED THESE TWO LINES FOR AUTHENTICATION
auth_basic "No Access";
auth_basic_user_file /etc/nginx/.htpasswd; 
 #server_name example.com;
 location / {
      proxy_pass http://localhost:5469/;
      proxy_set_header Host $host;
 }
}
```

```Reloading nginx
sudo service nginx reload
```
Now your Ethereum RPC apis would be available via [if your server ip is 222.333.444.555

nginx user:MMxf9RMA
nginx Pass:DkfjW3FJSGLlBg0XvYrMudFDTCFszmpYYvkc

http://MMxf9RMA:DkfjW3FJSGLlBg0XvYrMudFDTCFszmpYYvkc@222.333.444.555:5469

## License

The go-ethereum library (i.e. all code outside of the `cmd` directory) is licensed under the
[GNU Lesser General Public License v3.0](https://www.gnu.org/licenses/lgpl-3.0.en.html),
also included in our repository in the `COPYING.LESSER` file.

The go-ethereum binaries (i.e. all code inside of the `cmd` directory) is licensed under the
[GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html), also
included in our repository in the `COPYING` file.
