# substrate-castor-all

1.substrate-castor

2.castor-events-listener

3.castor-server

4.dapp-ui


---------------------------


## 1.substrate-castor

A new SRML-based Substrate node, ready for hacking.

### TODO
---
#### anakornk
- [x] Node Creation and Ownership Transfer
- [x] Simulate GE creation, stake and invest
- [x] Multi-TCX per GE, Simulates TCX 5 steps
- [x] Add sourcing features for nodes

#### lignyxg
- [ ] Compute Voting Power
- [ ] Link with GE

##### optional
- [ ] GE Withdraw
- [ ] add security/conditional checks (esp. TCX)
---

#### Custom Types
```
{
  "ContentHash": "[u8; 32]",
  "NodeType": "u32",
  "Node": {
    "id": "ContentHash",
    "node_type": "NodeType",
    "sources": "Vec<ContentHash>"
  },
  "GeId": "u64",
  "ActionId": "u64",
  "TcxId": "u64",
  "GovernanceEntity": {
    "threshold": "u64",
    "min_deposit": "Balance",
    "apply_stage_len": "Moment",
    "commit_stage_len": "Moment"
  },
  "Challenge": {
    "amount": "Balance",
    "voting_ends": "Moment",
    "resolved": "bool",
    "reward_pool": "Balance",
    "total_tokens": "Balance",
    "owner": "AccountId"
  },
  "ChallengeId": "u64",
  "Listing": {
    "id": "ListingId",
    "node_id": "ContentHash",
    "amount": "Balance",
    "application_expiry": "Moment",
    "whitelisted": "bool",
    "challenge_id": "ChallengeId",
    "owner": "AccountId"
  },
  "ListingId": "u64",
  "Poll": {
    "votes_for": "Balance",
    "votes_against": "Balance",
    "passed": "bool"
  },
  "Tcx": {
    "tcx_type": "u64"
  },
  "TcxType": "u64",
  "Link": {
    "source": "u32",
    "target": "u32"
  },
  "Like": {
    "from": "AccountId",
    "to": "ContentHash"
  },
  "Admire": {
    "from": "AccountId",
    "to": "ContentHash"
  },
  "Grant": {
    "from": "AccountId",
    "to": "ContentHash",
    "amount": "Balance"
  },
  "Report": {
    "from": "AccountId",
    "target": "ContentHash",
    "reason": "ContentHash"
  },
  "VecContentHash": "Vec<ContentHash>",
  "ReasonHash": "ContentHash",
  "AdmireId": "u64",
  "GrantId": "u64",
  "LikeId": "u64",
  "ReportId": "u64",
  "Quota": "u64",
  "ActionPoint": "Balance",
  "Energy": "Balance",
  "Reputation": "Balance"
}
```


### Build

Install Rust:

```bash
curl https://sh.rustup.rs -sSf | sh
```

Install required tools:

```bash
./scripts/init.sh
```

Build Wasm and native code:

```bash
cargo build
```

### Run

#### Single node development chain

You can start a development chain with:

```bash
cargo run -- --dev
```

Detailed logs may be shown by running the node with the following environment variables set: `RUST_LOG=debug RUST_BACKTRACE=1 cargo run -- --dev`.

#### Multi-node local testnet

If you want to see the multi-node consensus algorithm in action locally, then you can create a local testnet with two validator nodes for Alice and Bob, who are the initial authorities of the genesis chain that have been endowed with testnet units.

Optionally, give each node a name and expose them so they are listed on the Polkadot [telemetry site](https://telemetry.polkadot.io/#/Local%20Testnet).

You'll need two terminal windows open.

We'll start Alice's substrate node first on default TCP port 30333 with her chain database stored locally at `/tmp/alice`. The bootnode ID of her node is `QmRpheLN4JWdAnY7HGJfWFNbfkQCb6tFf4vvA6hgjMZKrR`, which is generated from the `--node-key` value that we specify below:

```bash
cargo run -- \
  --base-path /tmp/alice \
  --chain=local \
  --alice \
  --node-key 0000000000000000000000000000000000000000000000000000000000000001 \
  --telemetry-url ws://telemetry.polkadot.io:1024 \
  --validator
```

In the second terminal, we'll start Bob's substrate node on a different TCP port of 30334, and with his chain database stored locally at `/tmp/bob`. We'll specify a value for the `--bootnodes` option that will connect his node to Alice's bootnode ID on TCP port 30333:

```bash
cargo run -- \
  --base-path /tmp/bob \
  --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/QmRpheLN4JWdAnY7HGJfWFNbfkQCb6tFf4vvA6hgjMZKrR \
  --chain=local \
  --bob \
  --port 30334 \
  --telemetry-url ws://telemetry.polkadot.io:1024 \
  --validator
```

Additional CLI usage options are available and may be shown by running `cargo run -- --help`.


---------------------------


## 2.castor-events-listener
- Store castor network events to MongoDB.

### Setup
```
1. yarn install
```

### Run
```
1. start mongodb daemon
2. run substrate castor
3. yarn start
```

### .env Configuration
```
# MongoDB config
MONGO_HOST=127.0.0.1
MONGO_PORT=27017
MONGO_DB=castor_events
MONGO_COLLECTION=event_data

# Substrate Node Config
SUBSTRATE_HOST=localhost
SUBSTRATE_PORT=9944
SUBSTRATE_EVENT_SECTIONS=all
# SUBSTRATE_EVENT_SECTIONS=balances,assets,token,kitties
```

### mongdo - view all events
```
mongo
use castor_events
db.event_data.find()
```

#### TODO
- Maintain block height, deal with sync issues.
- Integer might overflow in mongodb.
- check if challenging multiple times might cause bug

#### Notes
- uses string for integers that do not need to be compared


---------------------------


## 3.castor-server
- Restful api for castor.network
- relies on castor-event-listener

### Setup
```
yarn install
```

### Run
```
1. Start MongoDB
2. Run Castor Event Listener
3. Run substrate castor
4. yarn start (default port: 7000)
```

#### Rest API
```
# nodes
/api/v1/nodes
/api/v1/nodes/:id

# ges
/api/v1/ges
/api/v1/ges/:id
/api/v1/ges/:id/tcxs
/api/v1/ges/:id/tasks

# tcx
/api/v1/tcxs
/api/v1/tcxs/:id
/api/v1/tcxs/:id/tasks

# account
/api/v1/accounts
/api/v1/accounts/:id
/api/v1/accounts/:id/nodes
/api/v1/accounts/:id/ges
/api/v1/accounts/:id/tasks

# node explorer
/api/v1/explorer/:id
```


---------------------------


## 4.dapp-ui

A.T.Dot dapp ui.

### Run
```
1. ipfs daemon
2. start substrate chain
3. start castor server
4. yarn start
```

#### TODO
##### Editor
- prevent duplicate links
- add arrow head to line links with css
- add features to remove and delete links

### 注意
```
对于account相关，默认使用账户 5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty
/api/v1/accounts/5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty/tasks
/api/v1/accounts/5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty/nodes
```