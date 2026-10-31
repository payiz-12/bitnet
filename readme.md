# Bitnet Core: Your Gateway to Decentralized, Programmable Money

Bitnet combines the decentralization of Bitcoin with the programmability of Ethereum, making it a dual-purpose blockchain technology. Utilize the protocol to create, transfer, and receive Bitnets, or leverage its smart contract capabilities to build complex applications—from digital assets and non-fungible tokens (NFTs) to decentralized autonomous organizations (DAOs) and much more.

> **Insight**: *The only way forward is through decentralization.*

## Community Build

This branch is an independent, community-maintained build of the Bitnet node:
the official v0.0.5 code with security fixes, working bootnodes and other
improvements. It follows the same consensus rules as the official client and
uses the same data folder, so the two run side by side on the network and you
can switch in either direction.

- [Installing or switching](docs/install.md)
- [What changed](docs/release-notes.md)
- [Running a public RPC endpoint](docs/public-rpc.md)
- [Adding Bitnet to a mining pool](docs/pool-operators.md)
- [Reproducible builds](docs/reproducible-builds.md)

## Node Not Syncing?

The usual reason is that the node finds no peers: none of the bootnodes in the
official v0.0.5 code answers any more. This build has working bootnodes. Switch
to it as described in the [install guide](docs/install.md); your `bitnet.db`
folder keeps working, so there is no need to delete it and sync again.

Start the node from its folder with `Bitnet.sh` (or `Bitnet.cmd` on Windows).
The scripts register the genesis block before starting the node, as the official
package does:

```bash
./bitnet --datadir bitnet.db init .genesis && ./bitnet --networkid 210 --config .nodeconfig
```

If it still does not sync, ask in the community channels; the community is
most active on [Discord](https://discord.com/invite/dtw7rKQfRs). Problems with
this build can be reported in its [issues](https://github.com/payiz-12/bitnet/issues).

## Install Script

You can install all the necessary dependencies by running:
```shell
python install.py
```

Replace `python` with `python3` if your system is version-sensitive.

## Build From Source

Should you wish to build Bitnet from source, ensure you have the following software:

- [Python](https://www.python.org/downloads/)
- [Go](https://go.dev/dl/) (installed by `install.py`); releases are built and tested with Go 1.27.3
- [GCC](https://gcc.gnu.org/releases.html) (installed by `install.py`)

> **Note**: If either of these components is missing or corrupt, the build will fail. For a pre-built version with binaries compatible with your operating system, visit the [Releases Page](https://github.com/payiz-12/bitnet/releases).

---

You can run the following script to build Bitnet from source:
```shell
python build.py
```

Replace `python` with `python3` if your system is version-sensitive.

Alternatively you can bypass the Python build script by using Go and GCC directly:
```shell
go run build/ci.go install ./cmd/bitnet
```

## Key Metrics

- **Block Reward**: 1
- **Uncle Reward**: 0.875 down to 0.25 by depth (1 to 6 blocks); see the [pool guide](docs/pool-operators.md)
- **Pre-mined Coins**: None
- **Total Supply**: ∞
- **Consensus Mechanism**: Proof of Work
- **Algorithm**: Ethash
- **Target Block Time**: 15 seconds
- **Smart Contracts**: Supported

## Genesis Block

The Mainnet Genesis Hash is as follows:

```
0xa3cc7f928cebbc82a199e3c506104df317244e5de86018b8753ef3096f674f1a
```
