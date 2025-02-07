---
tags: [Nodes]
description: Node Bootstrap is the process where a node *securely* downloads linear chain blocks to recreate the latest state of the chain locally. Bootstrapping a node is a multi-step process which requires downloading the chains required by the Primary Network (that is, the C-Chain, P-Chain, and X-Chain), as well as the chains required by any additional Subnets that the node explicitly tracks.
sidebar_label: "Bootstrapping: What to Expect"
pagination_label: What to Expect While Bootstrapping
sidebar_position: 0
---

# Node Bootstrap

Node Bootstrap is the process where a node *securely* downloads linear chain
blocks to recreate the latest state of the chain locally.

Bootstrap must guarantee that the local state of a node is in sync with the
state of other valid nodes. Once bootstrap is completed, a node has the latest
state of the chain and can verify new incoming transactions and reach consensus
with other nodes, collectively moving forward the chains.

Bootstrapping a node is a multi-step process which requires downloading the
chains required by the Primary Network (that is, the C-Chain, P-Chain, and
X-Chain), as well as the chains required by any additional Subnets that the node
explicitly tracks.

This document covers the high-level technical details of how bootstrapping
works. This document glosses over some specifics, but the
[AvalancheGo](https://github.com/ava-labs/avalanchego) codebase is open-source
and is available for curious-minded readers to learn more.

## Validators and Where to Find Them

Bootstrapping is all about downloading all previously accepted containers
*securely* so a node can have the latest correct state of the chain. A node
can't arbitrarily trust any source - a malicious actor could provide malicious
blocks, corrupting the bootstrapping node's local state, and making it
impossible for the node to correctly validate the network and reach consensus
with other correct nodes.

What's the most reliable source of information in the Avalanche ecosystem? It's
a *large enough* majority of validators. Therefore, the first step of
bootstrapping is finding a sufficient amount of validators to download
containers from.

The P-Chain is responsible for all platform-level operations, including staking
events that modify a Subnet's validator set. Whenever any chain (aside from the
P-Chain itself) bootstraps, it requests an up-to-date validator set for that
Subnet (Primary Network is a Subnet too). Once the Subnet's current validator
set is known, the node can securely download containers from these validators to
bootstrap the chain.

There is a caveat here: the validator set must be *up-to-date*. If a
bootstrapping node's validator set is stale, the node may incorrectly believe
that some nodes are still validators when their validation period has already
expired. A node might unknowingly end up requesting blocks from non-validators
which respond with malicious blocks that aren't safe to download.

**For this reason, every Avalanche node must fully bootstrap the P-chain first
before moving on to the other Primary Network chains and other Subnets to
guarantee that their validator sets are up-to-date**.

What about the P-chain? The P-chain can't ever have an up-to-date validator set
before completing its bootstrap. To solve this chicken-and-egg situation the
Avalanche Foundation maintains a trusted default set of validators called
beacons (but users are free to configure their own). Beacon Node-IDs and IP
addresses are listed in the [AvalancheGo
codebase](https://github.com/ava-labs/avalanchego/blob/master/genesis/beacons.go).
Every node has the beacon list available from the start and can reach out to them
as soon as it starts.

Validators are the only sources of truth for a blockchain. Validator
availability is so key to the bootstrapping process that **bootstrapping is
blocked until the node establishes a sufficient amount of secure connections to
validators**. If the node fails to reach a sufficient amount within a given
period of time, it shuts down as no operation can be carried out safely.

## Bootstrapping the Blockchain

Once a node is able to discover and connect to validator and beacon nodes, it's
able to start bootstrapping the blockchain by downloading the individual
containers.

One common misconception is that Avalanche blockchains are bootstrapped by
retrieving containers starting at genesis and working up to the currently
accepted frontier.

Instead, containers are downloaded from the accepted frontier downwards to
genesis, and then their corresponding state transitions are executed upwards
from genesis to the accepted frontier. The accepted frontier is the last
accepted block for linear chains.

Why can't nodes simply download blocks in chronological order, starting from
genesis upwards? The reason is efficiency: if nodes downloaded containers
upwards they would only get a safety guarantee by polling a majority of
validators for every single container. That's a lot of network traffic for a
single container, and a node would still need to do that for each container in
the chain.

Instead, if a node starts by securely retrieving the accepted frontier from a
majority of honest nodes and then recursively fetches the parent containers from
the accepted frontier down to genesis, it can cheaply check that containers are
correct just by verifying their IDs. Each Avalanche container has the IDs of its
parents (one block parent for linear chains)
and an ID's integrity can be guaranteed cryptographically.

Let's dive deeper into the two bootstrap phases - frontier retrieval and
container execution.

### Frontier Retrieval

The current frontier is retrieved by requesting them from validator or beacon
nodes. Avalanche bootstrap is designed to be robust - it must be able to make
progress even in the presence of slow validators or network failures. This
process needs to be fault-tolerant to these types of failures, since
bootstrapping may take quite some time to complete and network connections can
be unreliable.

Bootstrap starts when a node has connected to a sufficient majority of validator
stake. A node is able to start bootstrapping when it has connected to at least
$75\%$ of total validator stake.

Seeders are the first set of peers that a node reaches out to when trying to
figure out the current frontier. A subset of seeders is randomly sampled from
the validator set. Seeders might be slow and provide a stale frontier, be
malicious and return malicious container IDs, but they always provide an initial
set of candidate frontiers to work with.

Once a node has received the candidate frontiers form its seeders, it polls
**every network validator** to vet the candidates frontiers. It sends the list
of candidate frontiers it received from the seeders to each validator, asking
whether or not they know about these frontiers. Each validator responds
returning the subset of known candidates, regardless of how up-to-date or stale
the containers are. Each validator returns containers irrespective of their age
so that bootstrap works even in the presence of a stale frontier.

Frontier retrieval is completed when at least one of the candidate frontiers is
supported by at least $50\%$ of total validator stake. Multiple candidate
frontiers may be supported by a majority of stake, after which point the next
phase, container fetching starts.

At any point in these steps a network issue may occur, preventing a node from
retrieving or validating frontiers. If this occurs, bootstrap restarts by
sampling a new set of seeders and repeating the bootstrapping process,
optimistically assuming that the network issue will go away.

### Containers Execution

Once a node has at least one valid frontiers, it starts downloading parent
containers for each frontier. If it's the first time the node is running, it
won't know about any containers and will try fetching all parent containers
recursively from the accepted frontier down to genesis (unless [state
sync](#state-sync) is enabled). If bootstrap had already run previously,
some containers are already available locally and the node will stop as soon as
it finds a known one.

A node first just fetches and parses containers. Once the chain is complete, the
node executes them in chronological order starting from the earliest downloaded
container to the accepted frontier. This allows the node to rebuild the full
chain state and to eventually be in sync with the rest of the network.

## When Does Bootstrapping Finish?

You've seen how [bootstrap works](#bootstrapping-the-blockchain) for a single
chain. However, a node must bootstrap the chains in the Primary Network as well
as the chains in each Subnet it tracks. This begs the questions -
when are these chains bootstrapped? When is a node done bootstrapping?

The P-chain is always the first to bootstrap before any other chain. Once the
P-Chain has finished, all other chains start bootstrapping in parallel,
connecting to their own validators independently of one another.

A node completes bootstrapping a Subnet once all of its corresponding chains
have completed bootstrapping. Because the Primary Network is a special case of
Subnet that includes the entire network, this applies to it as well as any other
manually tracked Subnets.

Note that Subnets bootstrap is independently of one another - so even if one Subnet
has bootstrapped and is validating new transactions and adding new containers,
other Subnets may still be bootstrapping in parallel.

Within a single Subnet however, a Subnet isn't done bootstrapping until the last
chain completes bootstrapping. It's possible for a single chain to effectively
stall a node from finishing the bootstrap for a single Subnet, if it has a
sufficiently long history or each operation is complex and time consuming. Even
worse, other Subnet validators are continuously accepting new transactions and
adding new containers on top of the previously known frontier, so a node that's
slow to bootstrap can continuously fall behind the rest of the network.

Nodes mitigate this by restarting bootstrap for any chains which is blocked
waiting for the remaining Subnet chains to finish bootstrapping. These chains
repeat the frontier retrieval and container downloading phases to stay
up-to-date with the Subnet's ever moving current frontier until the slowest
chain has completed bootstrapping.

Once this is complete, a node is finally ready to validate the network.

## State Sync

The full node bootstrap process is long, and gets longer and longer over time as
more and more containers are accepted. Nodes need to bootstrap a chain by
reconstructing the full chain state locally - but downloading and executing each
container isn't the only way to do this.

Starting from [AvalancheGo version
1.7.11](https://github.com/ava-labs/avalanchego/releases/tag/v1.7.11), nodes can
use state sync to drastically cut down bootstrapping time on the C-Chain.
Instead of executing each block, state sync uses cryptographic techniques to
download and verify just the state associated with the current frontier. State
synced nodes can't serve every C-chain block ever historically accepted, but
they can safely retrieve the full C-chain state needed to validate in a much
shorter time. State sync will fetch the previous 256 blocks prior to support the previous block 
hash operation code.

State sync is currently only available for the C-chain. The P-chain and X-chain
currently bootstrap by downloading all blocks. Note that irrespective of the
bootstrap method used (including state sync), each chain is still blocked on all
other chains in its Subnet completing their bootstrap before continuing into
normal operation.

:::note

There are no configs to state sync an archival node. If you need all the historical state then
 you must not use state sync and setup the config of the node for an archival node.

:::

## Conclusions and FAQ

If you got this far, you've hopefully gotten a better idea of what's going on
when your node bootstraps. Here's a few frequently asked questions about
bootstrapping.

### How Can I Get the ETA for Node Bootstrap?

Logs provide information about both container downloading and their execution
for each chain. Here is an example

```text
[02-16|17:31:42.950] INFO <P Chain> bootstrap/bootstrapper.go:494 fetching blocks {"numFetchedBlocks": 5000, "numTotalBlocks": 101357, "eta": "2m52s"}
[02-16|17:31:58.110] INFO <P Chain> bootstrap/bootstrapper.go:494 fetching blocks {"numFetchedBlocks": 10000, "numTotalBlocks": 101357, "eta": "3m40s"}
[02-16|17:32:04.554] INFO <P Chain> bootstrap/bootstrapper.go:494 fetching blocks {"numFetchedBlocks": 15000, "numTotalBlocks": 101357, "eta": "2m56s"}
...
[02-16|17:36:52.404] INFO <P Chain> queue/jobs.go:203 executing operations {"numExecuted": 17881, "numToExecute": 101357, "eta": "2m20s"}
[02-16|17:37:22.467] INFO <P Chain> queue/jobs.go:203 executing operations {"numExecuted": 35009, "numToExecute": 101357, "eta": "1m54s"}
[02-16|17:37:52.468] INFO <P Chain> queue/jobs.go:203 executing operations {"numExecuted": 52713, "numToExecute": 101357, "eta": "1m23s"}
```

Similar logs are emitted for X and C chains and any chain in explicitly tracked
Subnets.

### Why Chain Bootstrap ETA Keeps On Changing?

As you saw in the [bootstrap completion
section](#when-does-bootstrapping-finish), a Subnet like the Primary Network
completes once all of its chains finish bootstrapping. Some Subnet chains may
have to wait for the slowest to finish. They'll restart bootstrapping in the
meantime, to make sure they won't fall back too much with respect to the network
accepted frontier.

### Why Are AvalancheGo APIs Disabled During Bootstrapping?

AvalancheGo APIs are [explicitly
disabled](https://github.com/ava-labs/avalanchego/blob/master/api/server/server.go#L367:L379)
during bootstrapping. The reason is that if the node has not fully rebuilt its
Subnets state, it can't provide accurate information. AvalancheGo APIs are
activated once bootstrap completes and node transition into its normal operating
mode, accepting and validating transactions.
