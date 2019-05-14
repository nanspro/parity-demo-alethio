# Visualizing and Monitoring blockchain using Alethio's tools
This tutorial aims to introduce people to alethio's tools such as "ethereum-lite-explorer" and "ethstats-network-dashboard" to help them visualize blockchain data with a clean and beautiful UI. We'll be integrating a block explorer and a network monitor dashboard for our private network created using any RPC enable client. In this tutorial we'll be using parity-ethereum to setup a local private network.

After going through this tute any user should be able to setup a node and also deploy a network of nodes locally (in this case parity) and will be able to connect block explorer and network monitoring tools with it.  

**Prerequisites**<br/>
To run this tutorial, you must have the following installed:

- Linux
- Git command line
- Docker and Docker-compose

## Setting up parity Node
First let's see how to set up a private node, we'll be building parity-ethereum from source for this

### Building from source

Clone parity-ethereum from [here](https://github.com/paritytech/parity-ethereum)
```
git clone https://github.com/paritytech/parity-ethereum
cd parity-etherum
```
**Install the prerequisites to run parity node as described [here](https://github.com/paritytech/parity-ethereum#build-dependencies)**

To include rust in your path variable of current shell do <br/>
`source $HOME/.cargo/env`

Run `cargo build --release --features final` you'll get a release file in `./target/release`

To see various flags and options to start a node <br/> 
run `./target/release/parity --help`

Lets **start** now by running<br/> 
`./target/release/parity --light --jsonrpc-interface all --ws-interface all  --ws-hosts all --ws-origins all`

### Using Docker 
Run `docker run -d --restart always --name parity-light -p 127.0.0.1:8545:8545 parity/parity:stable --light --jsonrpc-interface all` to download parity image locally and start it.


## Setting up a private chain using couple of nodes

### Using Docker
Clone [this](https://github.com/nanspro/parity-demo-alethio) repo to instantly setup a network consisting of 3 authorities and 3 members.
```
git clone https://github.com/nanspro/parity-demo-alethio
cd parity-demo-alethio
```

**Starting Network** <br/>
`docker-compose up -d`

_Note_: If use get some version error, then use parity v2.5.0 and then<br/>
`export PARITY_VERSION=v2.5.0`

And now our network is live on "localhost:8545"

#### Accounts
There is already an account with an empty password that has enough ether:

```
0x6B0c56d1Ad5144b4d37fa6e27DC9afd5C2435c3B
```

And another who is broke:
```
0x00E3d1Aa965aAfd61217635E5f99f7c1e567978f
```

You may also want to change the list of prefunded accounts in `parity/config/chain.json`.

Add JSON-formatted ethereum accounts to `parity/keys` and they will appear in the UI.

#### Access JSON RPC 
Talk to JSON RPC at [http://127.0.0.1:8545](http://127.0.0.1:8545) with your favorite client.

Be kind and send the poor an ether!

```
curl --data '{"jsonrpc":"2.0","method":"personal_sendTransaction","params":[{"from":"0x6B0c56d1Ad5144b4d37fa6e27DC9afd5C2435c3B","to":"0x00E3d1Aa965aAfd61217635E5f99f7c1e567978f","value":"0xde0b6b3a7640000"}, ""],"id":0}' -H "Content-Type: application/json" -X POST localhost:8545
```

## Connecting with Block Explorer
We'll use alethio's lite-explorer to view our network blocks with an amazing UI

On another terminal clone the **ethereum lite explorer**
```
git clone git@github.com:Alethio/ethereum-lite-explorer.git
cd ethereum-lite-explorer
```

Install dependencies<br/>
`npm install`

Build<br/>
`npm run build`

Copy the default config into dev config like this `cp config.default.json config.dev.json`<br/>
Open the config.dev.json and change the `APP_NODE_URL = 'http://127.0.0.1:8545'` to connect to your network.

Run `npm start` now and in a new tab a explorer will open showing block stats for the network like this

![Home](./static/home.png)

You can view our the transaction done using curl earlier in the blocks

![Block](./static/block.png)


## Connecting with EthStats Dashboard
We'll use alethio's **ethstats dashboard** to monitor and analyze our private network with an amazing UI

### EthStats Server
A network monitoring server from which our dashboard will fetch data.

```
git clone https://github.com/Alethio/ethstats-network-server
cd ethstats-network-server
```

We can run the server normally or in lite-mode, for this tutorial we will use **lite-mode** for a quick and smooth setup.<br/>
There are 2 ways to start the server in lite mode.

- Memory/No persistence - in case of a crash/restart the gathered data is lost.
- Redis/With persistence - in case of a crash/restart the gathered data is persisted into Redis.

```
cd docker/lite-mode/{choice_to_start}
docker-compose up
```

Running server in lite-mode with or without persistence pulls some dependencies and runs them in background
- **ethstats dashboard**: Front-end application for the EthStats Network Statistics tool  
- Third party dependency **deepstream**: A new type of open source server that syncs data and sends events across millions of clients in real time 

If we are running the server with persistence then our `docker-compose` will pull one more dependency 

- **Redis**: It is an open source, in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists etc.

### EthStats-CLI

The application connects to your Ethereum node through RPC and extract data that will be sent to the EthStats Server for analytics purposes. To use the cli first a node is required to register.

We'll be using docker to pull and run
```
mkdir config
docker run -d \
--restart always \
--net host \
-v /home/config/:/root/.config/configstore/ \
alethio/ethstats-cli --register --account-email your@email.com --node-name your_node_name --server-url http://localhost:3000
```

CLI Options

```
--server-url The url where our network-server is running
--account-email Your email account (you can use it later to sign in to https://net.ethstats.io)
--node-name name of your node
```

_Note_: The app is configured by default to connect to the Ethereum node on your local host (http://localhost:8545). To connect to a node running on a different host use flag `--client-url`.

Now we have ethstats-cli running in a container and is ready to send data, we'll need to start our EthStats Server to fetch data from this CLI.

### Accessing EthStats Dashboard

Now that our server is up and running along with EthStats CLI, it will fetch the data from CLI and pass it to dashboard, dashboard is also running in parallel so just go to "http://localhost:8080" and you can see our dashboard analyzing our private network like this.

![dashboard](./static/dashboard.png)


_Note: You can change the ports for running different components in `docker-compose.yml`, also read more about third party dependencies like [deepstream](https://deepstreamhub.com/open-source/), [redis](https://redis.io/) and see how they works_. 


## License

MIT &copy; [Alethio](https://aleth.io)



