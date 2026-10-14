# Cadcoin and the Bitcoin whitepaper

Cadcoin is a fork of Bitcoin Core **v31.1** that aims to be as faithful a copy of
Bitcoin as possible while running as its own, separate network.

* Original whitepaper: Satoshi Nakamoto, *Bitcoin: A Peer-to-Peer Electronic Cash System* (2008) —
  <https://bitcoin.org/bitcoin.pdf>
* Upstream source: <https://github.com/bitcoin/bitcoin> (tag `v31.1`)
* Language: C++ (same as Bitcoin Core), MIT licence.

## Whitepaper sections → code

| Whitepaper section | What it describes | Where it lives in this tree |
|---|---|---|
| 1. Introduction | Trustless electronic payments, no double spending without a trusted third party | Whole node: `src/validation.cpp`, `src/net_processing.cpp` |
| 2. Transactions | Chain of digital signatures; coins transfer by signing hash of previous tx + next owner's key | `src/primitives/transaction.h`, `src/script/interpreter.cpp` (signature checking), `src/script/sign.cpp`, `src/secp256k1/` (ECDSA/Schnorr on secp256k1) |
| 3. Timestamp Server | Hash of a block of items, each timestamp includes the previous one | `src/primitives/block.h` (`CBlockHeader::hashPrevBlock`, `nTime`), `src/chain.h` (`CBlockIndex`), median-time-past checks in `ContextualCheckBlockHeader` (`src/validation.cpp`) |
| 4. Proof-of-Work | SHA-256 hash with required leading zero bits; nonce; difficulty adjusted by a moving average to target blocks per hour | `src/pow.cpp` (`CheckProofOfWork`, `GetNextWorkRequired`, `CalculateNextWorkRequired`: 2016-block / two-week retarget, 10-minute spacing), `src/crypto/sha256*.cpp`, `src/hash.h` (double SHA-256), `src/arith_uint256.*` (compact `nBits` target) |
| 5. Network | Broadcast txs, miners collect into blocks, find PoW, broadcast; longest (most-work) chain wins | `src/net.cpp`, `src/net_processing.cpp` (P2P), `src/node/miner.cpp` (block assembly), `src/validation.cpp` (`ActivateBestChain`, chain selection by `nChainWork`) |
| 6. Incentive | First tx of a block is a coinbase creating new coins; later transaction fees; fixed supply | `GetBlockSubsidy` in `src/validation.cpp` (50 coins halving every 210,000 blocks), `src/consensus/amount.h` (`MAX_MONEY` = 21,000,000), coinbase checks in `src/consensus/tx_check.cpp` / `ConnectBlock` |
| 7. Reclaiming Disk Space | Merkle tree so spent transactions can be pruned | `src/consensus/merkle.cpp` (`BlockMerkleRoot`), pruning in `src/node/blockstorage.cpp` (`-prune`) |
| 8. Simplified Payment Verification | Clients keep only headers and Merkle branches | Headers-first sync (`src/headerssync.cpp`), `gettxoutproof`/`verifytxoutproof` RPCs and `src/merkleblock.cpp`, BIP157/158 compact filters in `src/blockfilter.cpp` |
| 9. Combining and Splitting Value | Multiple inputs and up to two (or more) outputs | `CTransaction` `vin`/`vout` in `src/primitives/transaction.h`; input/output sum checks in `src/consensus/tx_verify.cpp` |
| 10. Privacy | Public keys kept anonymous; new key pair per transaction | HD wallets (BIP32) in `src/wallet/`, descriptor wallets `src/script/descriptor.cpp`, address encoding `src/key_io.cpp` |
| 11. Calculations | Attacker catch-up probability | Not code; motivates confirmation counts. Reorg handling: `src/validation.cpp` (`ActivateBestChainStep`, `DisconnectTip`) |
| 12. Conclusion | — | — |

Later protocol upgrades inherited unchanged from Bitcoin (all active from block 1 on
Cadcoin): P2SH (BIP16), height in coinbase (BIP34), CHECKLOCKTIMEVERIFY (BIP65),
strict DER (BIP66), CSV/relative lock-time (BIP68/112/113), SegWit (BIP141/143/147),
Taproot/Tapscript (BIP340-342).

## Consensus parameters (unchanged from Bitcoin)

| Parameter | Value | Code |
|---|---|---|
| Max supply | 21,000,000 CBC | `MAX_MONEY` in `src/consensus/amount.h` |
| Initial subsidy | 50 CBC | `GetBlockSubsidy` (`src/validation.cpp`) |
| Halving interval | 210,000 blocks | `consensus.nSubsidyHalvingInterval` (`src/kernel/chainparams.cpp`) |
| Block target spacing | 600 s | `consensus.nPowTargetSpacing` |
| Retarget timespan | 14 days (2016 blocks) | `consensus.nPowTargetTimespan` |
| PoW | SHA-256d, `powLimit` = `00000000ffff…` | `src/pow.cpp` |
| Genesis difficulty | `0x1d00ffff` | `src/kernel/chainparams.cpp` |
| Smallest unit | 1 satoshi = 10⁻⁸ CBC | `COIN` in `src/consensus/amount.h` |
| Block weight limit, script rules, sigops, coinbase maturity (100) | identical | `src/consensus/consensus.h`, `src/script/` |

## Exactly what differs from Bitcoin Core v31.1

### Network identity (makes Cadcoin a separate network)

| | main | testnet (v3) | testnet4 | signet | regtest |
|---|---|---|---|---|---|
| Message start (magic) | `fc c4 b7 d2` | `0c 14 0b 07` | `1d 17 3e 29` | first 4 bytes of SHA256d(`"cadcoin"` ‖ challenge) | `fb c0 b6 db` |
| P2P port | 8533 | 18533 | 48533 | 38533 | 18544 |
| RPC port | 8532 | 18532 | 48532 | 38532 | 18543 |
| Tor onion target port | 8534 | 18534 | 48534 | 38534 | 18545 |
| Base58 P2PKH / P2SH / WIF | 28 (`C…`) / 87 (`c…`) / 156 | 127 (`t…`) / 130 (`u…`) / 255 | same as testnet | same as testnet | same as testnet |
| Bech32/Bech32m HRP | `cbc` | `tcbc` | `tcbc` | `tcbc` | `rcbc` |
| BIP32 xpub/xprv versions | unchanged (`xpub`/`xprv`, `tpub`/`tprv`) | | | | |

### Genesis blocks

All genesis blocks use the same construction as Bitcoin's (`CreateGenesisBlock`, 50-coin
unspendable coinbase, version 1) with a new timestamp message and an all-zero
(provably unspendable) output public key, as Bitcoin's testnet4 genesis does.

Message (main/testnet3/signet/regtest): `Cadcoin 14/Oct/2026 A peer-to-peer electronic cash system, faithful to Bitcoin`
Message (testnet4): `Cadcoin testnet4 14/Oct/2026`

| Network | nTime | nNonce | nBits | Block hash | Merkle root |
|---|---|---|---|---|---|
| main | 1791936000 | 2446764487 | 0x1d00ffff | `00000000c56558257c17b888ce5a03a49a00b7736f34cd5b66f73ffe31e932c4` | `85cea7ff70f38e646e6481c75eccb7c1198c7bc599bbcd82fa2eb6436d33ac39` |
| testnet | 1791936602 | 3531885399 | 0x1d00ffff | `0000000079250ddfae13f1869031b364544fb9ca14d5508c7f4851606f3d262d` | `85cea7ff70f38e646e6481c75eccb7c1198c7bc599bbcd82fa2eb6436d33ac39` |
| testnet4 | 1791937200 | 3446964782 | 0x1d00ffff | `0000000002676be9e9929baf95a2fa4f0309892b4fa1735c4fba6940b25e1ad0` | `72c6f7ebdbc7d13a624a4e080fff8c8263e6ee57aa13840de734f46c056a1a78` |
| signet | 1791936000 | 892069 | 0x1e0377ae | `0000026c4868b8ade8692cc76a7aa86de0848d765cd639e7df7c9e5c6f76fa2f` | `85cea7ff70f38e646e6481c75eccb7c1198c7bc599bbcd82fa2eb6436d33ac39` |
| regtest | 1791936000 | 1208 | 0x207fffff | `007d6550d95c58049917ca8e83af0717a03d7821a1f1f5d693e2510c07ddc101` | `85cea7ff70f38e646e6481c75eccb7c1198c7bc599bbcd82fa2eb6436d33ac39` |

Mainnet genesis was mined at Bitcoin's real starting difficulty `0x1d00ffff`.

### Chain parameters reset for a new chain

* DNS seeds and fixed seeds removed on all networks (none exist yet). Use `-addnode`/`-connect`/`-seednode`.
* `nMinimumChainWork`, `defaultAssumeValid`, `m_assumeutxo_data`, and `chainTxData` reset to empty/zero
  on main, testnet, testnet4 and signet (Bitcoin's values refer to Bitcoin blocks). Bitcoin Core v31
  has no checkpoints to remove.
* BIP16 / Taproot `script_flag_exceptions` (which name specific historical Bitcoin blocks) removed.
* Buried soft forks BIP34/BIP65/BIP66/CSV/SegWit active from height 1 on every network
  (BIP34 cannot apply to genesis; Bitcoin regtest's `SegwitHeight = 0` is kept on regtest).
  Taproot is `ALWAYS_ACTIVE` on every network. This is the same rule set Bitcoin enforces today;
  only the historical activation heights differ.
* Default signet challenge is `OP_TRUE` (`51`) since there is no Cadcoin signet operator; run a
  private signet with `-signetchallenge=<script>`. Signet message-start bytes are derived from
  `"cadcoin"` ‖ challenge so a Cadcoin signet never shares magic with a Bitcoin signet.
* `m_assumed_blockchain_size` / `m_assumed_chain_state_size` set to 1 GB.

### Branding

* Product name `Cadcoin Core` (`CLIENT_NAME`), user agent `/Cadcoin:31.1.0/`.
* Executables: `cadcoind`, `cadcoin-cli`, `cadcoin-tx`, `cadcoin-wallet`, `cadcoin-util`, `cadcoin-qt`,
  `cadcoin-chainstate`, `cadcoin` (wrapper), `test_cadcoin`, `bench_cadcoin`. CMake *target* names are
  unchanged from upstream; only `OUTPUT_NAME` differs (see end of top-level `CMakeLists.txt`).
* Data dir `~/.cadcoin` (`Cadcoin` on Windows/macOS), config `cadcoin.conf`, pid file `cadcoind.pid`.
* Currency unit `CBC` (GUI: CBC, mCBC, µCBC (bits), sat).
* RPC/CLI help strings mention Cadcoin/cadcoind/cadcoin-cli where they referred to Bitcoin binaries.
* C++ identifiers, file names, namespaces, internal comments and links to upstream pull requests/issues
  are intentionally left untouched so upstream changes can be merged easily.

### Tests

See the "Test status" section below.

## Test status

Pending.
