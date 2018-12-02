# Running a Lightning node

# this tutorial is provided as learning purpose
Nothing special
Please only run, this in public internet, if you know what you are doing

# requirements
* mainnet 250go
* testnet 25go
* debian stretch
* bitcoin - v0.16.3
* lightningd - v0.6.2

# basis

## install some tools
```bash
apt update && apt upgrade
apt-get install git build-essential htop tmux -y
```

## linux
here, we use debian. There is no perfect distribution. Debian as a good security focus and stable environment.

When interacting with this tutorial. Your default user should not be *root*.

## firewall
* blocks all ports except SSH/bitcoind(8333)/lightningd
* temporary open git/apt/80/443

Excellent tutorial are available in the web.

# first - bitcoin

## impl
Bitcoin as 3 majors implementations for server
* bitcoind maintain by bitcoin core developers and bitcoin developers
* btcd maintain by The btcsuite developers
* libbitcoin

We use bitcoind

## deps version
https://github.com/bitcoin/bitcoin/blob/master/doc/dependencies.md

## install required package
```bash
apt update
apt install autoconf libboost-all-dev libssl-dev libtool libevent-dev libminiupnpc-dev pkg-config -y
```

### building & install libdb
this is the library that store chain history.

```bash
wget 'http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz'
tar -xzvf db-4.8.30.NC.tar.gz
cd db-4.8.30.NC/build_unix/
../dist/configure --prefix=/usr/local --enable-cxx
make
sudo make install
```

### building & install protobuf
### install via apt should be v3.0.0
```bash
apt-get install libprotobuf-dev protobuf-compiler
```
or
### if you want exact version
```bash
git clone https://github.com/google/protobuf.git
cd protobuf
git checkout v2.6.1
git submodule update --init --recursive
./autogen.sh
./configure
make
make check
sudo make install
sudo ldconfig # refresh shared library cache for linux.
```

## exports for bitcoind compilation
```bash
export BDB_INCLUDE_PATH="/usr/local/BerkeleyDB.4.8/include"
export BDB_LIB_PATH="/usr/local/BerkeleyDB.4.8/lib"
ln -s /usr/local/BerkeleyDB.4.8/lib/libdb-4.8.so /usr/local/lib/libdb-4.8.so
```

# compiling bitcoind
```bash
git clone https://github.com/bitcoin/bitcoin.git
cd bitcoin
git checkout v0.16.3
./autogen.sh
./configure CXXFLAGS="--param ggc-min-expand=1 --param ggc-min-heapsize=32768" --enable-cxx --disable-shared --with-pic --without-gui --prefix=/usr/local/ LDFLAGS="-L$BDB_LIB_PATH -L/usr/local/lib -L." CPPFLAGS="-I$BDB_INCLUDE_PATH -I/include/google/protobuf"
make
sudo make install
```

without => self explaining
enable-cxx => because client have c++ code
disable shared => as you like if you prefer portable or shared libs
with-pic => position independant code
prefix => where the binary is installed

## create user
```bash
adduser bitcoin
```

## create your datadir
target a place with sufficient space, here we just do it in /opt
```bash
mkdir -p /opt/bitcoin-data
chown bitcoin:bitcoin /opt/bitcoin-data
```

## dropin your bitcoin.conf

Look at bitcoin.conf, rpc access is not setup.
Make sure you have rpc/zmq interface and txindex=1.
Copy bitcoin.conf in the data directory.

```bash
cp bitcoin.conf /opt/bitcoin-data
```

### generate rpc access

From the bitcoin repository's root, run the following command (replacing `{{login-name}}` with the login name of your choice):
```bash
./share/rpcauth/rpcauth.py {{login-name}}
```
This script will gave you the rpc access line to replace in bitcoin.conf and give you password to be used to connect to rpc socket.

### launch bitcoin daemon - bitcoind
as user bitcoin, launch bitcoin daemon

```bash
su bitcoin
bitcoind -datadir=/opt/bitcoin-data
```
or
```bash
bitcoind -datadir=/opt/bitcoin-data -testnet=1
```
for testnet


### look at logs - debug.log

```bash
tail -f /opt/bitcoin-data/debug.log
```

### interacting with bitcoin command line interface - bitcoin-cli

```bash
bitcoin-cli -datadir=/opt/bitcoin-data getblockcount
```

# second - Lightning

## new deps to install

```bash
sudo apt-get install -y \
  libgmp-dev \
  libsqlite3-dev python python3 net-tools zlib1g-dev
```

## first we clone the project

```bash
git clone https://github.com/ElementsProject/lightning.git
cd lightning
git checkout v0.6.2
```

## then build it
```bash
make
sudo make install
```

```bash
chown -R bitcoin:bitcoin /opt/clightning/
```

# run lightningd in testnet

```bash
su bitcoin
/usr/local/bin/lightningd --pid-file=/run/clightningd/lightningd.pid \
           --bitcoin-cli=/usr/local/bin/bitcoin-cli --bitcoin-datadir=/opt/bitcoin-data \
           --addr="{{ip_addr}}:9735" --alias={{super_alias}} \
           --lightning-dir=/opt/clightning/ --log-level=debug --network=testnet
```

# interacting with lightning-cli

```bash
lightning-cli --lightning-dir=/opt/bitcoin-data getinfo
```

# what next ?
* systemd
* tor
* rate limiting
* lightning charge
* enjoy
