# bitcoind-lightning-compose

# A docker-compose that builds a self validating bitcoind backend with a lightning (lnd) node.

# If you are tired going through tutorials to connect bitcoind to lightning daemon then you are at the right place.

# The configurations are stored either in:

* docker-compose.yml
* bitcoin.conf
* lnd.conf

### Uses a self validating v.23.0 bitcoind image as backend built from [ruimarinho's docker-bitcoin-core](https://github.com/ruimarinho/docker-bitcoin-core/tree/master/23)

### Lightning service uses the [lnd's image](https://github.com/lightningnetwork/lnd)

### Do you have a running bitcoin node with data already? You can use it as a backend for your lightning node by mounting the data directory to the btc service in the docker-compose.yml file.

# Configuration details (How the services are wired)

## bitcoin.conf

* `rpcuser` and `rpcpassword` - to authenticate RPC calls. (Define your own user and password)
* `rpcallowip=0.0.0.0/0`, `rpcport=8332`, `rpcbind=0.0.0.0` - from which ip addresses, through what port and on what
  address should bitcoind listen
  for RPC calls (Port is the default one, it binds to every network interface, and accepts connections from any ip as
  well)
* `zmq*` block and transaction broadcasting configuration (port:ip)

## lnd.conf

### LND uses to connect to the bitcoind backend:

* `bitcoin.node=bitcoind` use bitcoind as a backend
* `bitcoind.rpchost=btc` the hostname where bitcoind listens, this is the name of the service in the compose file
* `bitcoind.rpcuser=foo`, `bitcoind.rpcpass=your_password` should be the same as in the bitcoin.conf file
* `bitcoind.zmqpubrawblock=tcp://btc:28332`, `bitcoind.zmqpubrawtx=tcp://btc:28333` should be the same as in the
  bitcoin.conf file
* `bitcoind.dir=/root/.bitcoin` a directory containing bitcoin data, it will be mounted here via the compose file

### To connect to our lightning node (via ZAP):

* `tlsextraip=YOUR_IP_ADDRESS` the ip of your node (with port forward you can reach it from outside of your network, in
  that case your router's ip comes here)
* `tlsextradomain=YOUR_IP_ADDRESS` same as above but with domain names
* `rpclisten=0.0.0.0:10009` tells LND to accept connections on all external interfaces on the default port (10009)

## docker-compose.yml

### btc service

* `environment: - BITCOIN_DATA=/home/bitcoin/.bitcoin` where to store the bitcoin data inside the container
* `volumes: - ./bitcoin.conf:/home/bitcoin/.bitcoin/bitcoin.conf` where to mount the bitcoin.conf file inside the
  container
* `volumes: - /your/path/to/bitcoin_data:/home/bitcoin/.bitcoin` where to store the bitcoin data directory on the
  host machine

### lnd service

* `volumes: - ./lnd.conf:/root/.lnd/lnd.conf` where to mount the lnd.conf file inside the container
* `volumes: - /your/path/to/bitcoin_data:/root/.bitcoin/` where to mount the bitcoin data directory inside the
  container
* `volumes: - /your/path/to/lnd:/root/.lnd/` where to store the lnd data directory on the host machine
* `command:--noseedbackup` to disable the seed backup prompt for testing purposes (remove it if you want to be prompted
  for the seed backup)

### +1 lndconnect service in a separate docker-compose file ( lndconnect/docker-compose.yml )
#### `lndconnect` is a service that generates a connection string for you to connect to your node via ZAP
* `volume: - /your/path/to/lnd:/root/.lnd/` where to mount the lnd data directory inside the container
* `--host=YOUR_IP_ADDRESS` since we run it in a container we need to specify the host ip address to correctly generate the
  connection string

# How to use it
* `docker-compose up -d` to start the services, wait a for the bitcoind to sync with the blockchain (it can take a
  while)
* `docker-compose -f lndconnect/docker-compose.yml up` to start the lndconnect service (it will generate a connection
  string for you to connect to your node via ZAP) (you can issue this command before the bitcoind syncs with the blockchain)
