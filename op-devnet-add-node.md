
# Add an OP Node to an OP Devnet

The target is to setup an op-node (with op-geth) in an running local OP Stack devnet environment to be able to sync data from other op-nodes.

## Software Dependencies

| Dependency | Version | Version Check Command        |
|------------|---------|------------------------------|
| git        | ^2      | `git --version`              |
| go         | ^1.21   | `go version`                 |
| node       | ^20     | `node --version`             |
| foundry    | ^0.2.0  | `forge --version`            |
| make       | ^3      | `make --version`             |
| jq         | ^1.6    | `jq --version`               |
| direnv     | ^2      | `direnv --version`           |
| docker     | ^27     | `docker --version`			  |

## Running an OP Devnet

To start a local OP Stack devnet including L1 + L2, enter the Optimism monorepo and execute the following command:

```bash
make devnet-up
```

The following processes will be started:

- Container ops-bedrock-l1-1
- Container ops-bedrock-l1-bn-1
- Container ops-bedrock-l1-vc-1
- Container ops-bedrock-l2-1
- Container ops-bedrock-artifact-server-1
- Container ops-bedrock-op-node-1
- Container ops-bedrock-op-batcher-1
- Container ops-bedrock-op-proposer-1
- Container ops-bedrock-op-challenger-1

The following steps will create and start up an op-node manually to join the local devnet.

## Starting OP Geth

Initiate the OP Geth process:

```bash
git clone https://github.com/ethereum-optimism/op-geth.git
cd op-geth
git checkout v1.101408.0

make geth

./build/bin/geth init --state.scheme=hash --datadir=datadir ../optimism/.devnet/genesis-l2.json

cp ../optimism/ops-bedrock/test-jwt-secret.txt jwt.txt

./build/bin/geth \
  --datadir ./datadir \
  --http \
  --http.corsdomain="*" \
  --http.vhosts="*" \
  --http.addr=0.0.0.0 \
  --http.port=5545 \
  --http.api=web3,debug,eth,txpool,net,engine \
  --ws \
  --ws.addr=0.0.0.0 \
  --ws.port=5546 \
  --ws.origins="*" \
  --ws.api=debug,eth,txpool,net,engine \
  --syncmode=full \
  --gcmode=archive \
  --nodiscover \
  --maxpeers=0 \
  --networkid=901 \
  --port=30503 \
  --authrpc.vhosts="*" \
  --authrpc.addr=0.0.0.0 \
  --authrpc.port=5551 \
  --authrpc.jwtsecret=./jwt.txt \
  --rollup.disabletxpoolgossip=true
```
Notice the ports are modified to avoid conflicts.

## Starting the OP Node

To start the op-node, enter the Optimism monorepo and execute the following commands:

```bash
make op-node

./op-node/bin/op-node \
  --l2=http://localhost:5551 \
  --l2.jwt-secret=./ops-bedrock/test-jwt-secret.txt \
  --sequencer.enabled=false \
  --verifier.l1-confs=4 \
  --rollup.config=.devnet/rollup.json \
  --rpc.addr=0.0.0.0 \
  --rpc.port=9845 \
  --rpc.enable-admin \
  --p2p.listen.ip=0.0.0.0 \
  --p2p.listen.tcp=9004 \
  --p2p.listen.udp=9004 \
  --p2p.static=/ip4/127.0.0.1/tcp/9003/p2p/16Uiu2HAmHqrXGts25TtKMBRHtvhWZLNypsobKoggpZye1XQtJpbZ \
  --l1=$L1_RPC_URL \
  --l1.beacon=http://localhost:5052 \
  --l1.rpckind=$L1_RPC_KIND
```
Note 
1. Sequencer is disabled and p2p is enabled to sync with peer.
2. To find a peer id from L2:
```
cast rpc opp2p_self --rpc-url http://localhost:7545
```
Use the correct peerID as the value of `--p2p.static` to start op-node to sync data.

## References

 - Follow the basic steps outlined in the [OP Stack Tutorial](https://docs.optimism.io/builders/chain-operators/tutorials/create-l2-rollup). 
 - Qi's tutorial video: [Zoom Recording](https://us02web.zoom.us/rec/share/6YQE-khKf-cQpCIsaNUw8gxCMhDByCxxfFmvE4lszAqqWJJOI9H8tQwWQoHYDHfV.xWqob3trZCXGVnqV).
 - [Configuration options](https://docs.optimism.io/builders/node-operators/configuration/consensus-config)
