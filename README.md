Cadcoin Core
============

Cadcoin is a faithful fork of [Bitcoin Core](https://github.com/bitcoin/bitcoin)
**v31.1**, written in C++ and released under the MIT licence (see [COPYING](COPYING)).

It implements the system described in Satoshi Nakamoto's whitepaper,
[*Bitcoin: A Peer-to-Peer Electronic Cash System*](https://bitcoin.org/bitcoin.pdf),
with **identical consensus rules** to Bitcoin:

- 21,000,000 coin supply cap (50 coin initial subsidy, halving every 210,000 blocks)
- 10 minute block target, difficulty retarget every 2016 blocks
- SHA-256d proof of work, initial difficulty `0x1d00ffff`
- Same script system, SegWit, Taproot, etc. (all active from block 1)

It is a **separate network**: it has its own genesis blocks, network magic,
ports, address prefixes and no DNS/fixed seeds yet. Cadcoin nodes will not talk
to Bitcoin nodes and Cadcoin addresses are not valid Bitcoin addresses.

See [docs/whitepaper.md](docs/whitepaper.md) for a mapping of the whitepaper to
the code and the complete list of differences from Bitcoin Core.

| | |
|---|---|
| Ticker / unit | **CBC** (mCBC, µCBC, satoshi) |
| Binaries | `cadcoind`, `cadcoin-cli`, `cadcoin-qt`, `cadcoin-tx`, `cadcoin-wallet`, `cadcoin-util`, `cadcoin` |
| Data directory | `~/.cadcoin` (Linux), `~/Library/Application Support/Cadcoin` (macOS), `%LOCALAPPDATA%\Cadcoin` (Windows) |
| Config file | `cadcoin.conf` |
| Mainnet P2P / RPC port | 8533 / 8532 |
| Bech32 HRP | `cbc` (main), `tcbc` (testnet/testnet4/signet), `rcbc` (regtest) |

Building
--------

Build instructions are the same as upstream Bitcoin Core (CMake). On Debian/Ubuntu:

```sh
sudo apt install build-essential cmake pkgconf python3 libevent-dev libboost-dev libsqlite3-dev
cmake -B build -DBUILD_GUI=OFF
cmake --build build -j"$(nproc)"
ctest --test-dir build -j"$(nproc)"
```

See [doc/build-unix.md](doc/build-unix.md) and the other `doc/build-*.md` files.
CMake target names are unchanged from upstream (e.g. `bitcoind`), only the
produced executables are renamed (e.g. `build/bin/cadcoind`).

Quick start (regtest)
---------------------

```sh
build/bin/cadcoind -regtest -daemon
build/bin/cadcoin-cli -regtest createwallet test
build/bin/cadcoin-cli -regtest -generate 101
build/bin/cadcoin-cli -regtest getbalance
```

Upstream
--------

All credit for the software goes to the Bitcoin Core developers. The original
Bitcoin Core README is kept at
[doc/README-bitcoin-core-upstream.md](doc/README-bitcoin-core-upstream.md).
References to `github.com/bitcoin/bitcoin` pull requests and issues in the source
and docs point to upstream history and are intentionally left unchanged.
