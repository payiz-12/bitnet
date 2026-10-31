# Bitnet community build: release notes

## First release

An independent build of the Bitnet node, based on the official v0.0.5 code
(commit b929d15, which reports itself as 0.0.4-stable). The consensus rules are
unchanged: same blocks, rewards and difficulty, no fork. Official nodes and this
build accept each other's blocks, and both use the same `bitnet.db` folder, so
you can switch in either direction. How to install or switch: `docs/install.md`
in the repository, also included in every package as `INSTALL.md`.

### Security fixes

The first four are backported from go-ethereum, each with a test that fails on
the official code.

- **Ping flood (CVE-2023-40591).** Every received ping started a new goroutine,
  so a flood of pings could exhaust memory. Pongs are now sent from one loop.
- **Zero-count header requests (CVE-2024-32972).** A request for 0 headers
  wrapped around and read headers back to genesis. It now gets an empty answer,
  and header reads from the freezer are capped at 2 MB.
- **ECIES and secp256k1 (GHSA-mr7q-c9w9-wh4h, GHSA-2gjw-fg97-vg3r,
  GHSA-m6j8-rg6r-7mv8).** A short ciphertext could crash a node during the
  connection handshake, and points off the curve could leak bits of the node
  key. Create a new node key after switching: delete `bitnet.db/geth/nodekey`
  before the next start.
- **Oversized responses (CVE-2026-26313).** Responses were decoded in full
  before anyone checked that they had been asked for; a 2 MB message made the
  node allocate 146 MB, and a peer could repeat it until memory ran out.
  Responses are now matched to their request and counted before decoding.
- **Current Go.** The official v0.0.5 binaries are built with Go 1.22.2, and
  some nodes still run builds made with Go 1.19 or 1.20; none of these get
  security fixes any more. This build uses Go 1.27.3 (the memsize debug page,
  which does not link with current Go, was removed).

### Finding peers

- **Working bootnodes.** None of the eight bootnodes in the official code
  answers any more, so a new node found no peers unless given one by hand. The
  bootnodes published on bitnetmoney.com replace them; a new node now finds its
  peers within seconds. `.nodeconfig` also lists the GTPool node, which accepts
  connections but has no discovery port.
- **Bitnet peers only.** The node no longer reads Ethereum's DNS node list, and
  candidates from the discovery table, which all Ethereum-family networks
  share, are dialed only if they announce a compatible Bitnet fork ID.
- **Bitnet defaults.** The network id defaults to 210 (a node started without
  `--networkid` used to announce network 1 and could not join), the p2p port to
  30210 as in `.nodeconfig`, and the startup log names the Bitnet network.
  Ethereum leftovers are gone: the Rinkeby, Goerli and Sepolia presets, the
  proof-of-stake Engine API with its port 8551, the version check against
  Geth's vulnerability list, and the 4 GB cache default.

### Chain safety

- **Optional reorganisation limit.** `--reorg.maxdepth N` refuses, while the
  node is in sync, reorganisations that would drop more than N blocks, so a
  miner with most of the hashrate cannot replace past payments with a chain
  mined in private. Off by default; see `INSTALL.md` for the trade-off.
- **Replaced transactions.** `bitnet_subscribe("replacedTransactions")` reports
  when a pending transaction is replaced by another with the same nonce. A
  merchant who accepts payments before they are mined can see when one is being
  taken back (`samePayment` false).

### New read-only RPC methods

Enable them with `bitnet` in `--http.api`; the console has `bitnet.supply()`,
`bitnet.minerShares()`, `bitnet.confirmations()` and `bitnet.reorgs()`.

- `bitnet_supply(block?)`: BTN issued up to a block, counted by the node from
  block and uncle rewards, with the coins at the burn addresses and the
  circulating supply.
- `bitnet_minerShares(blocks?)`: who mined the newest blocks (1000 by default),
  the average block time and the Nakamoto coefficient.
- `bitnet_confirmations(risk?, blocks?)` and `bitnet_confirmationsFor(share,
  risk?)`: how many confirmations keep the chance of a reversed payment below
  `risk` (0.1% by default), using the formula from the Bitcoin paper.
- `bitnet_reorgs()`: the reorganisations the node made or refused since it
  started.

### RPC servers

- The node refuses to start when the admin, debug, personal or miner module is
  served over HTTP or WebSocket on an address other machines can reach;
  `--rpc.allow-unsafe-public` overrides this. Pools whose node RPC is reached
  from another machine with these modules need the flag.
- JSON-RPC batches are limited to 1000 calls and 25 MB of responses
  (`--rpc.batch-request-limit`, `--rpc.batch-response-max-size`).
- `docs/public-rpc.md` describes running a public endpoint behind nginx.

### Mining

- `eth_hashrate` was always 0, for CPU mining and for rates reported by remote
  miners; it now shows them.
- Tested on a private chain with the Bitnet rules: work from `eth_getWork`,
  forged solutions refused, valid ones accepted, 1 BTN per block plus fees, CPU
  mining with `--miner.threads`, and blocks accepted in both directions between
  this build and the official client.

### Builds and packages

- Release packages are laid out like the official package (`bitnet`,
  `.genesis`, `.nodeconfig`, start scripts), so an existing installation
  switches by replacing the program. The start scripts also work from another
  directory and pass extra flags to the node.
- The Linux binaries are reproducible: anyone can rebuild one from the source
  and get the same SHA256 (`docs/reproducible-builds.md`).
- A container image is published to ghcr.io; it uses `.nodeconfig` and port
  30210.

### How it was checked

- Synced the main network from genesis to the head, and separately imported
  the exported chain offline; both match the official chain at 22 checkpoints,
  including block hashes and state roots.
- `bitnet_supply` matched the supply shown by the BitnetScan explorer.
- One data folder was used by the official client, this build, the official
  client and this build again, with every head matching the main chain.
