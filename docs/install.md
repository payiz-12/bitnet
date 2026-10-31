# Installing the Bitnet community build

The community build is an independent build of the Bitnet node: the official
v0.0.5 code with security fixes, working bootnodes and the other changes listed
in the [release notes](release-notes.md). It follows the same consensus rules
as the official client, so both run side by side on the same network, and it
uses the same data folder, so you can switch in either direction.

## Download

Each release has one package per platform, laid out like the official package:

| Platform | Package |
|---|---|
| Windows (64-bit) | `bitnet-<version>-windows-amd64.zip` |
| Linux (x86-64) | `bitnet-<version>-linux-amd64.tar.gz` |
| Linux (ARM64, e.g. Raspberry Pi 4/5) | `bitnet-<version>-linux-arm64.tar.gz` |
| macOS (Apple silicon) | `bitnet-<version>-darwin-arm64.tar.gz` |
| macOS (Intel) | `bitnet-<version>-darwin-amd64.tar.gz` |

A package contains the `bitnet` program (`bitnet.exe` on Windows), the
`.genesis` and `.nodeconfig` files, the start scripts `Bitnet.sh` and
`BitnetConsole.sh` (`Bitnet.cmd` and `BitnetConsole.cmd` on Windows), this
guide and the license. The bare programs are attached as well.

Check a download against `SHA256SUMS` from the same release:

```bash
sha256sum bitnet-<version>-linux-amd64.tar.gz       # Linux
shasum -a 256 bitnet-<version>-darwin-arm64.tar.gz  # macOS
```

```powershell
Get-FileHash bitnet-<version>-windows-amd64.zip     # Windows PowerShell
```

The printed hash must equal the one on the file's line in `SHA256SUMS`. The
Linux programs can also be rebuilt from source to the same bytes; see
[reproducible builds](reproducible-builds.md).

## Switching from the official package

If you run the official package, your folder holds `bitnet` or `bitnet.exe`,
`.genesis`, `.nodeconfig`, the start scripts and the `bitnet.db` data folder.

1. Stop the node with Ctrl+C in its window and wait until it prints
   `Blockchain stopped`.
2. Back up `bitnet.db/keystore` if you keep accounts on the node. A copy of
   the whole `bitnet.db` folder lets you go back to exactly where you were.
3. Replace `bitnet` or `bitnet.exe` with the one from the package. Replace
   `.nodeconfig` and the start scripts too: the new `.nodeconfig` lists a
   peer that answers today, and the scripts work from any directory.
4. Delete `bitnet.db/geth/nodekey`, so that the node creates a new node key
   when it starts. The official client could leak bits of this key to a peer
   (one of the fixed security issues); the key only identifies the node on
   the network and holds no funds. If your node's address is published as a
   bootnode or static node, publish the new one, printed at start as `self=`.
5. Start the node as before, with `Bitnet.sh` or `Bitnet.cmd`.

The node continues from the blocks it already has; nothing is downloaded again.
To go back, put the official program back in the folder. Both clients read and
extend the same `bitnet.db`; this was tested by switching official, community,
official and community again on one data folder, with every head checked
against the main chain.

## New installation

Unpack the package and run the start script from its folder:

```bash
./Bitnet.sh      # Linux and macOS
```

```cmd
Bitnet.cmd       :: Windows
```

The script registers the genesis block and starts the node, which keeps its
data in `bitnet.db` next to the script. On macOS, if the system refuses to open
a downloaded program, allow it with `xattr -d com.apple.quarantine bitnet`.

The node finds its peers through the bootnodes, usually within seconds. A new
node then downloads the current state and the chain's headers, blocks and
receipts, about 8 million blocks and 4 GB (snap sync, the default). In a test on
a home connection in October 2026 it was in sync after 43 minutes; the speed
depends mostly on how fast the peers serve blocks. To execute every block from
the genesis block instead, add `--syncmode full`; that takes several hours.

Flags given to the start script are passed on to the node, for example
`./Bitnet.sh --http` to turn on the local RPC server.

## Ports

| Port | Use |
|---|---|
| 30210 TCP and UDP | connections to other nodes, and node discovery |
| 8545 TCP | HTTP-RPC, only with `--http`, on 127.0.0.1 by default |

A node behind a home router connects out to its peers without any setup. If you
can, forward port 30210 (TCP and UDP) to it: then other nodes can connect to
yours, which the network needs, because few Bitnet nodes accept connections
today. Never open the RPC port to the internet; see
[running a public RPC endpoint](public-rpc.md) for doing that safely.

## Console

`BitnetConsole.sh` or `BitnetConsole.cmd` opens a JavaScript console on the
running node:

```js
eth.blockNumber        // newest block
net.peerCount          // connected peers
admin.peers            // who they are
bitnet.supply()        // BTN issued so far
bitnet.minerShares()   // who mined the last 1000 blocks
bitnet.confirmations() // blocks to wait for a payment, given those shares
```

## Mining

The node gives work to mining programs and accepts their solutions through
`eth_getWork` and `eth_submitWork`. For solo mining, start it with your address
and the local RPC server:

```bash
./Bitnet.sh --mine --miner.threads 0 --miner.etherbase 0xYOUR_ADDRESS \
  --http --http.api eth,net,web3
```

`--miner.threads 0` leaves the CPU free and only serves work. A GPU miner that
speaks this interface, such as ethminer, connects with
`-P http://127.0.0.1:8545`. Most current GPU miners speak only Stratum and need
pool software in between.

A mining node keeps the Ethash dataset of the current and the next epoch on
disk, about 3.3 GB each at the current height, so allow for about 7 GB of free
disk space and enough memory. The rewards go to the address given; each block
pays 1 BTN, plus fees.

`--miner.threads N` mines on N CPU threads. It works, but a CPU has a tiny part
of the network's hashrate: a recent laptop at about 0.5 MH/s against a network
of about 46 GH/s finds a block every two to three weeks on average.

Pools that reach their node's RPC from another machine with the `personal` or
`miner` modules must add `--rpc.allow-unsafe-public`; see below.

## Docker

```bash
docker run -d --name bitnet -v bitnet-data:/data \
  -p 30210:30210 -p 30210:30210/udp ghcr.io/<owner>/bitnet:<version>
```

The image runs with the same `.nodeconfig`; flags after the image name are
passed on to the node. See [reproducible builds](reproducible-builds.md#running-the-image).

## Options of this build

| Flag | Default | Effect |
|---|---|---|
| `--reorg.maxdepth N` | 0 (off) | while in sync, refuse chain reorganisations that drop more than N blocks |
| `--rpc.batch-request-limit N` | 1000 | most calls in one JSON-RPC batch |
| `--rpc.batch-response-max-size N` | 25000000 | most bytes in one batch response |
| `--rpc.allow-unsafe-public` | off | allow the admin, debug, personal and miner modules over HTTP or WebSocket on a non-local address |

Without `--rpc.allow-unsafe-public` the node refuses to start when one of those
modules would be reachable from other machines, because they can move funds or
stop the node. Keep them on 127.0.0.1, or behind a firewall that only your own
machines pass, before turning the check off.

`--reorg.maxdepth` stops a miner with most of the hashrate from mining a long
chain in private and replacing payments the node has already accepted. It
applies only while the node's newest block is less than an hour old, so a node
that is catching up can always leave a stale branch. The cost: after a real
network split that lasts longer than the limit, a node on the lighter side does
not switch to the heavier side by itself; restart it without the flag to let it
follow. Leave it off unless you watch the node.

The read-only `bitnet` RPC module (`--http.api eth,net,web3,bitnet`) adds
`bitnet_supply`, `bitnet_minerShares`, `bitnet_confirmations`,
`bitnet_confirmationsFor`, `bitnet_reorgs` and, over WebSocket or IPC, the
`replacedTransactions` subscription; see the release notes.

## Building from source

With Go and a C compiler installed:

```bash
go run build/ci.go install ./cmd/bitnet
```

The program is written to `build/bin/bitnet`. Release builds use Go 1.27.3; see
[reproducible builds](reproducible-builds.md).
