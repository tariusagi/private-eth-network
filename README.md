# Set up a private Ethereum network
Guide to set up a private, clique Ethereum network for research and development. 

"Clique" means using Proof-of-Authority consensus mechanism built into Geth (Go Ethereum, an Ethereum node client program). PoA is chosen because it allows predefined validators, which is very suitable for a small, even single-node, network like in this guide. More on PoA [here](https://academy.binance.com/en/articles/proof-of-authority-explained).

Based on this [Private Networks](https://geth.ethereum.org/docs/interface/private-network) link.

*The following steps was done on a Raspberry Pi 4 (with 4GB RAM) board with a 16GB SD card.*

## Install Geth
Just follow this [Installing Geth](https://geth.ethereum.org/docs/install-and-build/installing-geth#install-on-ubuntu-via-ppas) link.

## Create an initial account
Run `geth account new` and enter your password, Geth should create a new account and output something like:

    ubuntu@pi4:~$ geth account new
    INFO [11-25|18:49:49.063] Maximum peer count                       ETH=50 LES=0 total=50
    INFO [11-25|18:49:49.063] Smartcard socket not found, disabling    err="stat /run/pcscd/pcscd.comm: no such file or directory"
    Your new account is locked with a password. Please give a password. Do not forget this password.
    Password:
    Repeat password:
    
    Your new key was generated
    
    Public address of the key:   0xCF6d4AeCc9ABE064C8f46115746115aa4967700c
    Path of the secret key file: /home/ubuntu/.ethereum/keystore/UTC--2021-11-25T11-50-16.555208031Z--cf6d4aecc9abe064c8f46115746115aa4967700c
    
    - You can share your public address with anyone. Others need it to interact with you.
    - You must NEVER share the secret key with anyone! The key controls access to your funds!
    - You must BACKUP your key file! Without the key, it's impossible to access account funds!
    - You must REMEMBER your password! Without the password, it's impossible to decrypt the key!


Remember the password, the public address and path of the secret key file, we will need them later.

## Configure the genesis block
Create the `~/.ethereum/genesis.json` with the following content:
```json
{
  "config": {
    "chainId": 7882,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "clique": {
      "period": 15,
      "epoch": 30000
    }
  },
  "difficulty": "1",
  "gasLimit": "8000000",
  "extradata": "0x0000000000000000000000000000000000000000000000000000000000000000CF6d4AeCc9ABE064C8f46115746115aa4967700c0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "alloc": {
    "0xCF6d4AeCc9ABE064C8f46115746115aa4967700c": { "balance": "10000000000000000000000" }
  }
}
```
In the above file, note these settings:
- `chainId` was set to `7882`. Actually this can be any number of your choice, but it's better to avoid using well-known chain's ID. A list of well-known chain' IDs can be found [here](https://chainlist.org/).
- `clique` consensus mechanism was set. Minimum difference between blocks' timestamps is 15s and epoch is set to 30000 blocks to mimic mainnet behavior (see [EIP-225](https://eips.ethereum.org/EIPS/eip-225)). These settings basically mean that a new block will be record every 15s, which looks like real-world mainnet. This is important because if transaction happens too quickly, the developers might not take into account the real user experience in real-world mainnet while designing the UI/UX for their apps.
- `difficulty` was set to `1`, so this node can easily find new blocks without taking too long to compute.
- `gasLimit` was set to `8000000`, that means this node will reject transaction that costs more than `8,000,000` gas.
- `extradata` include the intial account's public address (in between a series of 32 and 65 zeroes, without `0x` prefix).
- `balance` was set `10000000000000000000000` wei, which is `10,000` ethers.

To help with creating the `extradata`, place the address in between these line (without the `0x` prefix), and then join thems before putting the final text into the file:

    0x0000000000000000000000000000000000000000000000000000000000000000
    PUT THE ADDRESSE HERE WITHOUT THE 0x PREFIX
    0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000

## Initialize the Geth database
Run `geth init ~/.ethereum/genesis.json`. It should output something like:

    INFO [11-25|19:44:03.403] Maximum peer count                       ETH=50 LES=0 total=50
    INFO [11-25|19:44:03.404] Smartcard socket not found, disabling    err="stat /run/pcscd/pcscd.comm: no such file or directory"
    INFO [11-25|19:44:03.406] Set global gas cap                       cap=50,000,000
    INFO [11-25|19:44:03.406] Allocated cache and file handles         database=/home/ubuntu/.ethereum/geth/chaindata cache=16.00MiB handles=16
    INFO [11-25|19:44:03.429] Writing custom genesis block
    INFO [11-25|19:44:03.431] Persisted trie from memory database      nodes=1 size=149.00B time="353.014µs" gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
    INFO [11-25|19:44:03.433] Successfully wrote genesis state         database=chaindata                             hash=48fb8b..db389a
    INFO [11-25|19:44:03.434] Allocated cache and file handles         database=/home/ubuntu/.ethereum/geth/lightchaindata cache=16.00MiB handles=16
    INFO [11-25|19:44:03.461] Writing custom genesis block
    INFO [11-25|19:44:03.463] Persisted trie from memory database      nodes=1 size=149.00B time="296.607µs" gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
    INFO [11-25|19:44:03.465] Successfully wrote genesis state         database=lightchaindata                        hash=48fb8b..db389a

## Run the network
First, create the `~/.ethereum/password.txt` and put the password used to create the intial account there.

Then, run:

    geth --nodiscover --http --http.vhosts "*" --http.addr 0.0.0.0 --http.port 8545 --http.api admin,personal,eth,net,web3,txpool,miner,clique --mine --unlock CF6d4AeCc9ABE064C8f46115746115aa4967700c --password ~/.ethereum/password.txt --allow-insecure-unlock

Make sure to put the initial account address after the `--unlock` option, ưhich is `CF6d4AeCc9ABE064C8f46115746115aa4967700c` in this example.

That command will start an Ethereum network, which:
- Disable peers discovery. Peers must be added manually.
- Enable HTTP RPC service on port `8545` and listen on all interfaces.
- Allow RPC calls from all hosts.
- Enable these RPC APIs: `admin`, `personal`, `eth`, `net`, `web3`, `txpool`, `miner`, `clique`. Refer to [this](https://geth.ethereum.org/docs/rpc/server) link for the API documentation.
- Enable mining.
- Unlock the initial account using the password in the given file.

## Verify the network operation
To make sure the network run as expected, run `geth attach` to connect to the running network. The Geth's console should open like this:

    Welcome to the Geth JavaScript console!
    
    instance: Geth/v1.10.13-stable-7a0c19f8/linux-arm64/go1.17.2
    coinbase: 0xcf6d4aecc9abe064c8f46115746115aa4967700c
    at block: 129 (Thu Nov 25 2021 20:00:59 GMT+0700 (+07))
    datadir: /home/ubuntu/.ethereum
    modules: admin:1.0 clique:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0                                                                                                              rpc:1.0 txpool:1.0 web3:1.0
    
    To exit, press ctrl-d or type exit
    >

Make sure the coinbase address is the same as the initial account's public address. Now verify its balance with this command at the Geth JavaScript console:

```js
> eth.getBalance(eth.accounts[0], 'latest')
```

It should return `1e+22`, which is the balance of the account in wei, or `10,000` ethers.

To check if the initial account was unlocked and the network can create blocks for transactions, first get any account public address in MetaMask, and run the following command at the Geth console:
```js
> eth.sendTransaction({from:eth.accounts[0], to:"0xF9E8DC188Cba0Bb26130E811D0037fcE39F19A48", value: web3.toWei(100, "ether")}) 
```
That command should send 100 ethers from the intial account to the MetaMask account (which is `0xF9E8DC188Cba0Bb26130E811D0037fcE39F19A48` in this example). Now run:
```js
> eth.getBalance(eth.accounts[0], 'latest')
```
The result should be `9.9e+21` wei, which is `9,900` ethers (minus `100` spent ethers).

## Connect the MetaMask wallet
Open MetaMask wallet, and create a new network with the following parameters:

- Network Name: `My network` (or any other name).
- New RPC URL: `http://192.168.1.2:8545`.
- Chain ID: `7882`.

Remember to replace the node's IP with your own (`192.168.1.2` in this example).

After that, switch to this new network. The MetaMask's account which was used to receive the transaction from the network's initial account in the previous step should now show `100 ETH` in its balance.

## Import the initial account into MetaMask
To import the network's initial account into MetaMask, first we need its private key. We will create a Node program to do this.

Run the following commands at the shell to create a new Node project:
```sh
mkdir ethpk
cd ethpk
npm init
npm install keythereum
```
After that, edit the `package.json` file and add a script section like this:

```json
...
"scripts": {
  "start": "node index.js"
},
...
```

Now create the `index.js` file with this content:

```js
const keth = require("keythereum");
const DATADIR = `${process.env.HOME}/.ethereum/`;
const ADDRESS = "0xCF6d4AeCc9ABE064C8f46115746115aa4967700c";
const PASSWORD = "Hocnuahocmai!";

console.log("Extracting private key for " + ADDRESS + "...");
var keyObject = keth.importFromFile(ADDRESS, DATADIR);
var privateKey = keth.recover(PASSWORD, keyObject);
console.log("Private key:", privateKey.toString('hex'));
```
Remember to replace the `ADDRESS` and `PASSWORD` with your own.

Finally, run `npm start` and wait for a while. It should output the public address and the private key of the inifial account like this:

    Extracting private key for 0xCF6d4AeCc9ABE064C8f46115746115aa4967700c...
    Private key: 2b0a9b87abbe21a0637e1abaa1634a4a67569981e1eb5434a180b71d658f312e

Now that the private key was retrieved, import it into MetaMask and start using it from there.

## Useful Geth console commands
Running node's info:
```js
> admin.nodeInfo
```

`Clique` network status:
```js
> clique.status()
```

Current block number:
```js
> eth.blockNumber
8131
```
Get a block's info:
```js
> eth.getBlock(8131)
```

Get a list of accounts owned by running client:
```js
> eth.accounts
```

Get an account's balance:
```js
> web3.fromWei(eth.getBalance(ADDRESS, 'latest'), "ether")
```

Send some ethers (`SENDER` and `RECIPIENT` are addresses, `VALUE` is in ether):
```js
> eth.sendTransaction({from: SENDER, to: RECIPIENT, value: web3.toWei(VALUE, "ether")}) 
```

Get a transaction's details:
```js
> eth.getTransaction("0x79ac75f98630b3eba7ac44f207639865ce5b2adec674332bf74f22a29986a957")
```

Get all transactions' details:
```js
> Array(eth.blockNumber).fill().map((e, i) => i).filter(e => eth.getBlock(e).transactions.length ? true: false).map(e =>  eth.getBlock(e).transactions.map(e => eth.getTransaction(e)))
```

Get all transaction sent from ADDRESS:
```js
> Array(eth.blockNumber).fill().map((e, i) => i).filter(e => eth.getBlock(e).transactions.length ? true: false).map(e => eth.getBlock(e).transactions.map(e => eth.getTransaction(e)).filter(e => e.from == ADDRESS.toLowerCase()))
```

Get all transaction sent to ADDRESS:
```js
> Array(eth.blockNumber).fill().map((e, i) => i).filter(e => eth.getBlock(e).transactions.length ? true: false).map(e => eth.getBlock(e).transactions.map(e => eth.getTransaction(e)).filter(e => e.to == ADDRESS.toLowerCase()))
```

## Utilities
To aid working with the private network, use the following scripts:

`eth-init`: initialize or reset the network.
```bash
#!/bin/bash
echo "WARNING: this program will reset the Geth database!"
echo 'Will you proceed?'
select yn in "Cancel" "Proceed" ; do
    case $yn in
        Cancel ) exit;;
        Proceed ) break;;
    esac
done

echo "Reset and re-initialize Geth database..."
rm -fr ~/.ethereum/geth
geth init ~/.ethereum/genesis.json
```

`eth-start`: start or attach the network in a `tmux` session (`tmux` is required, can be installed with `sudo apt install tmux`).
```bash
#!/bin/bash
# Replace the following parameters if needed.
PORT=8545
ACC=0xCF6d4AeCc9ABE064C8f46115746115aa4967700c

echo Checking the Ethereum network...
sleep 1

if tmux has-session -t geth 2>/dev/null; then
        echo Network is already running, attaching now...
        tmux -2 attach-session -t geth:0.1
        exit 0
fi

echo Network is not running, now starting it...

# Create a new session named "geth".
tmux new-session -d -s geth

# Create the layout.
tmux split-window -v -t geth

# Now run the commands in panes.
# Top pannel: the Ethereum network.
tmux send-keys -t geth:0.0 "geth --nodiscover --http --http.vhosts '*' --http.addr 0.0.0.0 --http.port $PORT --http.api admin,personal,eth,net,web3,txpool,miner,clique --mine --unlock $ACC --password ~/.ethereum/password.txt --allow-insecure-unlock" Enter
# Wait 5s for the network to come up.
sleep 5
# Bottom pannel: the Geth console.
tmux send-keys -t geth:0.1 "geth attach" Enter

# Re-attach tmux session.
tmux -2 attach-session -t geth:0.1
```