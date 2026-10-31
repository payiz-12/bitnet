# Adding Bitnet to a mining pool

Bitnet (BTN) is a proof-of-work chain built on Geth 1.11 with standard Ethash.
Any pool that runs an Ethash coin through a Geth node (`eth_getWork` /
`eth_submitWork`) can run Bitnet the same way. This page lists what a pool
needs to know.

## Chain parameters

| | |
|---|---|
| Name, ticker | Bitnet, BTN |
| Algorithm | Ethash: epoch length 30,000 blocks, the Ethereum DAG schedule (not Etchash) |
| Chain id / network id | 210 / 210 (EIP-155 signatures use chain id 210) |
| Genesis hash | `0xa3cc7f928cebbc82a199e3c506104df317244e5de86018b8753ef3096f674f1a` |
| Block reward | 1 BTN, fixed, at every height; no halvings, no difficulty bomb |
| Uncle reward | (U + 8 − B) / 8 BTN for an uncle of height U in block B, at most 2 uncles per block |
| Nephew reward | 1/32 BTN per included uncle, to the block's miner |
| Transaction fees | all to the block's miner; no base fee is burned |
| Target block time | 15 seconds (15.2 s on average in October 2026) |
| Address format | 20-byte hex, `0x…`, as on Ethereum |
| EVM | Berlin; there is no London fork, so send legacy (gas price) or EIP-2930 transactions, not EIP-1559 ones |
| P2P port | 30210 TCP and UDP |

Uncle rewards by depth (B − U): 1 → 0.875, 2 → 0.75, 3 → 0.625, 4 → 0.5,
5 → 0.375, 6 → 0.25 BTN.

The DAG is about 3.3 GB at block 7.9 million (epoch 263) and grows by 8 MB
every 30,000 blocks, about every 5 days. GPUs with 4 GB of memory can mine
Bitnet until roughly 2028.

## Pool software

- **Reward.** Some pool software computes block rewards from Ethereum's
  schedule (5, 3 and 2 ETH at Ethereum's fork heights); open-ethereum-pool's
  block unlocker is an example. Set it to a constant 1 BTN, with the uncle and
  nephew rewards above, or miners will be credited the wrong amounts.
- **Network list.** Software that only knows a fixed list of Ethereum networks
  needs chain id 210 and the genesis hash above.
- **Calls used.** The node answers the usual Geth calls. The mining ones were
  tested on a private chain with the Bitnet rules: `eth_getWork` serves work
  that matches `eth_getBlockByNumber("pending")`, `eth_submitWork` refuses
  forged solutions and accepts valid ones, `eth_submitHashrate` is reported by
  `eth_hashrate`, and a payout sent with `eth_sendTransaction` was mined with
  its fee going to the miner.

## The node

Use the community build (`docs/install.md`, releases at
https://github.com/payiz-12/bitnet/releases) or the official v0.0.5 client.
The community build is patched against remote crashes and memory exhaustion,
which matters most for a pool, since every minute a pool node is down its
miners work on stale blocks.

A node that serves work to the pool:

```bash
./Bitnet.sh --mine --miner.threads 0 --miner.etherbase 0xPOOL_ADDRESS \
  --http --http.addr 127.0.0.1 --http.api eth,net,web3,bitnet
```

- `--miner.threads 0` serves work without mining on the CPU.
- The node keeps the Ethash dataset of the current and the next epoch on disk,
  about 7 GB in all at the current height.
- Keep the RPC server on 127.0.0.1 or behind a firewall. The community build
  refuses to start when the admin, debug, personal or miner module is reachable
  from other machines; `--rpc.allow-unsafe-public` overrides that check, for a
  pool whose software reaches the node from a separate, firewalled host.
- If payouts are signed by the node, unlocking an account while HTTP is on
  needs Geth's `--allow-insecure-unlock`; signing payouts outside the node and
  sending them with `eth_sendRawTransaction` is safer.
- Run the node with open port 30210 if you can. Few Bitnet nodes accept
  connections, and a pool node with many peers hears about new blocks sooner
  and loses fewer of its own as uncles.

## Confirmations

Uncles are common on Bitnet, so wait before crediting blocks as final. Many
Ethereum pools waited 120 blocks (about 30 minutes here). The community build
can work out the number from the real hashrate distribution:
`bitnet_confirmations` returns how many blocks keep the chance of a reversal
below 0.1% against the largest miner of the last 1000 blocks, and
`bitnet_minerShares` shows that distribution. `--reorg.maxdepth N` makes the
node refuse reorganisations deeper than N blocks while it is in sync; read the
trade-off in `docs/install.md` before using it.

## Links for the pool's coin page

| | |
|---|---|
| Website | https://bitnet.technology |
| Explorers | https://explorer.bitnetmoney.com, https://bitnetscan.vercel.app |
| Explorer paths (both) | `/block/<number>`, `/tx/<hash>`, `/address/<address>` |
| Source | https://github.com/BitnetMoney/bitnet (official), https://github.com/payiz-12/bitnet (community build) |
| Statistics | https://miningpoolstats.stream/bitnetmoney, https://whattomine.com/coins/378-btn-ethash |
| Community | Discord, Telegram (Bitnet_Money), X (@BitnetMoney) |

Once the pool runs, add it on MiningPoolStats with the "Missing pool?" link on
the Bitnet page, so that miners can find it.
