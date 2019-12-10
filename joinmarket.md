## JoinMarket guide for RaspiBolt

(this guide is unfinished draft, work in progress)

### Introduction

[JoinMarket](https://github.com/JoinMarket-Org/joinmarket-clientserver) is a CoinJoin software, which allows you to increase privacy and fungibility of on-chain Bitcoin transactions. It includes it's own Bitcoin wallet, backed by `bitcoind`, and uses market maker / market taker model, which means that either you pay small fee for having CoinJoin privacy fast (taker) or just keep software running and then you get paid for providing liquidity for CoinJoin's, in addition gaining privacy in a longer periods of time (maker). Even if you aren't interested in privacy of your coins, you can use JoinMarket for some little passive income from your bitcoins, without giving up your private keys. From my personal experience, currently earnings from running JoinMarket as a market maker (called yield generator bot) are bigger than what you get from routing Lightning Network payments (but still don't expect to get rich fast with that, will be less than 1% per year most likely).

### Preparations

#### Install dependencies

* With user "admin", install necessary dependencies

```
$ sudo apt-get install python-virtualenv curl python3-dev python3-pip build-essential automake pkg-config libtool libgmp-dev libltdl-dev libssl-dev libatlas3-base
```

#### Tor (optional)

It isn't strict requirement, but for the privacy it's recommended to use JoinMarket with Tor. Follow [RaspiBolt Tor guide](https://stadicus.github.io/RaspiBolt/raspibolt_69_tor.html) on how to install Tor on your RaspiBolt.

### Install JoinMarket

* Open a “bitcoin” user session and change into the home directory

`$ sudo su - bitcoin`

* Download, verify and extract the latest release (check the [Releases page](https://github.com/JoinMarket-Org/joinmarket-clientserver/releases) on Github for the correct links)

```
# download software
$ mkdir /home/bitcoin/download
$ cd /home/bitcoin/download
$ wget -O joinmarket-clientserver-0.6.1.tar.gz https://github.com/JoinMarket-Org/joinmarket-clientserver/archive/v0.6.1.tar.gz
$ wget https://github.com/JoinMarket-Org/joinmarket-clientserver/releases/download/v0.6.1/joinmarket-clientserver-0.6.1.tar.gz.asc

# verify that the release is signed by Adam Gibson (check the fingerprint)
# fingerprint should match https://github.com/JoinMarket-Org/joinmarket-clientserver/releases
$ wget https://raw.githubusercontent.com/JoinMarket-Org/joinmarket-clientserver/master/pubkeys/AdamGibson.asc
$ gpg --import ./AdamGibson.asc
gpg: keybox '/home/bitcoin/.gnupg/pubring.kbx' created
gpg: key 141001A1AF77F20B: 1 signature not checked due to a missing key
gpg: /home/bitcoin/.gnupg/trustdb.gpg: trustdb created
gpg: key 141001A1AF77F20B: public key "Adam Gibson (CODE SIGNING KEY) <ekaggata@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg: no ultimately trusted keys found
$ gpg --verify joinmarket-clientserver-0.6.1.tar.gz.asc
gpg: assuming signed data in 'joinmarket-clientserver-0.6.1.tar.gz'
gpg: Signature made Tue 10 Dec 2019 14:40:53 EET
gpg:                using RSA key 2B6FC204D9BF332D062B461A141001A1AF77F20B
gpg: Good signature from "Adam Gibson (CODE SIGNING KEY) <ekaggata@gmail.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 2B6F C204 D9BF 332D 062B  461A 1410 01A1 AF77 F20B
```

* Install JoinMarket
```
$ tar xvzf joinmarket-clientserver-0.6.1.tar.gz -C /home/bitcoin
$ cd /home/bitcoin
$ ln -s joinmarket-clientserver-0.6.1 joinmarket
$ cd joinmarket
$ ./install.sh --without-qt
```

### First run

```
$ source jmvenv/bin/activate
(jmvenv) $ cd scripts
(jmvenv) $ python wallet-tool.py
Created a new `joinmarket.cfg`. Please review and adopt the settings and restart joinmarket.
```

* Edit configuration file (`nano -w joinmarket.cfg`) and specify your bitcoind RPC settings. Optionally, if you have Tor enabled, comment out clearnet host entires and `socks5 = false` under `[MESSAGING:server1]` and `[MESSAGING:server2]` and uncomment the ones with `.onion` addresses and `socks5 = true` (example below is for Tor enabled configuration).
```
[BLOCKCHAIN]
#options: bitcoin-rpc, regtest
blockchain_source = bitcoin-rpc
network = mainnet
rpc_host = localhost
rpc_port = 8332
rpc_user = rcpuser
rpc_password = rpcpassword
rpc_wallet_file =

[MESSAGING:server1]
#host = irc.cyberguerrilla.org
channel = joinmarket-pit
port = 6697
usessl = true
#socks5 = false
socks5_host = localhost
socks5_port = 9050

#for tor
host = epynixtbonxn4odv34z4eqnlamnpuwfz6uwmsamcqd62si7cbix5hqad.onion
socks5 = true

[MESSAGING:server2]
#host = irc.darkscience.net
channel = joinmarket-pit
port = 6697
usessl = true
#socks5 = false
socks5_host = localhost
socks5_port = 9050

#for tor
host = darksci3bfoka7tw.onion
socks5 = true
```
### Generate JoinMarket wallet
```
(jvmenv) $ python wallet-tool.py generate
Would you like to use a two-factor mnemonic recovery phrase? write 'n' if you don't know what this is (y/n): n
Not using mnemonic extension
Enter wallet file encryption passphrase: 
Reenter wallet file encryption passphrase: 
Input wallet file name (default: wallet.jmdat): 
Write down this wallet recovery mnemonic

hour embark smile mansion wisdom rebel loud enhance clean man panel broccoli

Generated wallet OK
```
Write down the words, they will allow to recover wallet later on different machine in case of hardware failure or other problem.

### Fund JoinMarket wallet

JoinMarket wallet contains five separate sub-wallets (accounts) or pockets called "mixdepths". Idea is that coins between different mixdepths are never mixed together. When you do a CoinJoin transaction, change output goes back to the same mixdepth, but one of the equal amount outputs goes either to address of a different wallet (if you are taker) or to a different mixdepth in the same JoinMarket wallet (if you are a maker).

* Run `wallet-tool.py` default method to get list of addresses. Send some bitcoins to the first address of mixdepth 0. At first run it will give a message about need to restart JM and rescan blockchain. No need to rescan, just run command again.
```
$ python wallet-tool.py -m 0 wallet.jmdat
Enter wallet decryption passphrase: 
2019-11-10 18:57:09,377 [INFO]  Detected new wallet, performing initial import
restart Bitcoin Core with -rescan or use `bitcoin-cli rescanblockchain` if you're recovering an existing wallet from backup seed
Otherwise just restart this joinmarket application.
$ python wallet-tool.py -m 0 wallet.jmdat
Enter wallet decryption passphrase: 
2019-11-10 18:57:34,427 [INFO]  Detected new wallet, performing initial import
JM wallet
mixdepth        0       xpub6D52Hj7tztwBocht5MMAwW9nB4rD6KiFuqUtyR8Uzczna5m2TetrfHf5StgXQfp9n72SNKSwpMYYT7AzTTNds8yHpAyAwtzwgZkpG7yoNHs
external addresses      m/49'/0'/0'/0   xpub6EywwThpb4GJTPDM3eQw1RVAeKZpR5cFsEfmVWK1DXrtZHkds77t7ixs1SezycsAvnm1SyogyzxMtcxASy8TZTSzXY6sZH81QcXhoB4dRJH
m/49'/0'/0'/0/0         3BWEZFFjPcM2j9BjM6pQBYKqLZaVYxpyJF      0.00000000      new
m/49'/0'/0'/0/1         3Md9siWt7VJcgZfqyrQBNGHXmSpz2msUQY      0.00000000      new
m/49'/0'/0'/0/2         3DDf79zYDFX8WuVuMQAt2GWDKd4RGQX6dV      0.00000000      new
m/49'/0'/0'/0/3         36jar4xRAbPFTyqzV7RvYNpdmAtmwn9ENi      0.00000000      new
m/49'/0'/0'/0/4         3PfXrrLeNMifDuWv1KR4bz2p29XyoY6m8d      0.00000000      new
m/49'/0'/0'/0/5         3J5BsmfBtG2ULTPMZBC7vuNzRBCgWyaZ8x      0.00000000      new
Balance:        0.00000000
internal addresses      m/49'/0'/0'/1
Balance:        0.00000000
Balance for mixdepth 0: 0.00000000
Total balance:  0.00000000
```

### Running the yield generator bot

Yield generator is a maker bot that provides liquidity to the JoinMarket, so that others (takers) can make a CoinJoin's and they pay you a small fee for the service. It is recommended to fund your JoinMarket wallet with at least 0.1 BTC to run the yield generator. But the general principle is - the more funds you deposit in the wallet, the bigger chance of having passive CoinJoin transactions you have. But don't go reckless and remember that it is a hot wallet, so security is not the same as with a hardware wallet or other cold storage.

* Read the basics: https://github.com/JoinMarket-Org/joinmarket-clientserver/blob/master/docs/YIELDGENERATOR.md

* Look at the settings (`nano -w yg-privacyenhanced.py`) and change them if you want to. Defaults could be ok, but you could also lower minium CoinJoin transaction amount (`minsize`) to 100000 sats (0.001 BTC, default is 1000000 sats or 0.01 BTC). Also current relative CoinJoin maker fee (`cjfee_r`) default is 0.02%, you might want to rise it to 0.03%, as it is what Wasabi Wallet charges per anonimity set (`cjfee_r = 0.0003`). Note that values are approximations, yg-privacyenhanced will randomize them a little bit, due to privacy reasons.

* Run the yield generator
```
(jmvenv) $ python yg-privacyenhanced.py wallet.jmdat
```

#### Running the yield generator in background (after you close ssh connection to the RaspiBolt

* Install tmux from the "admin" user

`$ sudo apt-get install tmux`

* Start tmux from the "bitcoin" user

```
$ sudo su - bitcoin
$ tmux`
```
* Start yield generator inside tmux session
```
$ cd /home/bitcoin/joinmarket
$ source jmvenv/bin/activate
(jmvenv) $ cd scripts
(jmvenv) $ python yg-privacyenhanced.py wallet.jmdat
```
* Press Ctrl+B and then D to detach from tmux session (it will keep running in a background)

* Later you can attach to that session from "bitcoin" user

`$ tmux attach`

* Read more details about using tmux in this guide: https://www.ocf.berkeley.edu/~ckuehl/tmux/

### Sending payments

Note that you cannot use JoinMarket as a taker while yield generator is running with the same wallet. Before sending payments, you should stop yield generator, pressing Ctrl+C in a screen where it is running. So, idea is to stop yield generator, do your payment as a taker, and then start yield generator again. If you will try to send a payment while yield generator is running on the same wallet, you will get error that wallet is locked by another process.

Mixing maker and taker roles in a single wallet is actually good for your privacy too.

* See https://github.com/JoinMarket-Org/joinmarket-clientserver/blob/master/docs/USAGE.md#try-out-a-coinjoin-using-sendpaymentpy

### Checking wallet balance and history

* Summary of wallet balances: `python wallet-tool.py wallet.jmdat summary`

* Wallet transaction history: `python wallet-tool.py wallet.jmdat history -v 4`

### Running the tumbler

Tumbler is a program that do series of CoinJoin's with various amounts and timing between them, to completely break the link between different addresses.

* See https://github.com/JoinMarket-Org/joinmarket-clientserver/blob/master/docs/tumblerguide.md

### Other notes

* Every time you disconnect from the RaspiBolt and connect again, if you are in a fresh session, before running any JoinMarket commands, you need to do the following from "bitcoin" user:
```
$ cd /home/bitcoin/joinmarket
$ source jmvenv/bin/activate
(jmvenv) $ cd scripts
```

### How to upgrade

The latest release can be found on the Github page of the JoinMarket project. Make sure to read the Release Notes, as these can include important upgrade information. https://github.com/JoinMarket-Org/joinmarket-clientserver/releases

All this must be done from "bitcoin" user.

* Stop yield generator but, if it is running.

* Remove existing JoinMarket symlink: `unlink /home/bitcoin/joinmarket`

* Download, verify, extract and install the JoinMarket as described in the [Install JoinMarket](#install-joinmarket) section of this guide.

* Copy configuration and wallet file(s) from old JoinMarket directory to the new one (replace x.y.z with the version number of previous installed JoinMarket version, for example, 0.6.0):
```
$ cp /home/bitcoin/joinmarket-x.y.z/scripts/joinmarket.cfg /home/bitcoin/joinmarket/scripts/
$ cp /home/bitcoin/joinmarket-x.y.z/scripts/wallets/* /home/bitcoin/joinmarket/scripts/wallets/
```

### Useful links

* [JoinMarket docs](https://github.com/JoinMarket-Org/joinmarket-clientserver/tree/master/docs)
* [JoinMarket guide for RaspiBlitz](https://github.com/openoms/bitcoin-tutorials/blob/master/joinmarket/README.md)
* [Bitcoin privacy wiki](https://en.bitcoin.it/Privacy)
