# stratum-proxy

Universal Stratum Proxy for Bitcoin/Litecoin-forked Coins.

`stratum-proxy`is a small **developer-focused** TCP Stratum proxy, a helper tool for developers of miners or pools. It was created as a successor of the [rinpool-proxy](https://github.com/takologi/rinpool-proxy) project from the Rincoin project.

The proxy sits between a miner and a mining pool, forwards stratum traffic **byte-for-byte**, and writes a log that lets you see, replay, debug, and analyze what has been actually happening on the wire.

***Warning: do not use in production! This tool is intended for testing and development. It is not hardened, it logs sensitive traffic to disk, and it does not attempt to be robust against malicious inputs.***


## Features

- It is designed to work with **any** Bitcoin or Litecoin fork
- Uses the coin core's actual hash function (not a reimplementation)
- Automatically detects `GetPoWHash()` (altcoins) vs `GetHash()` (Bitcoin) at compile time
- TOML-based configuration with regex decorators for log coloring
- Zero manual hash function porting required
- TCP only (no TLS termination).
- only one single miner is expected (no concurrency)


## Quick Start

```bash
# 1. Configure the coin core (see COIN_CORES.md for examples)
cat > coin_core_repo.conf << 'EOF'
COIN_CORE_REPO_URL=https://github.com/bitcoin/bitcoin.git
COIN_CORE_BRANCH=v28.0
COIN_CORE_CONFIGURE_OPTS=--disable-wallet --disable-tests --disable-bench --disable-zmq --without-gui
EOF

# 2. Clone and build the coin core
make clone-core
make build-core

# 3. Build the proxy
make build

# 4. Configure and run
cp stratum-sample.toml stratum.toml
# Edit stratum.toml with your pool settings
./build/stratum-proxy stratum.toml
```

## How It Works

1. You specify the git URL of your target coin's core repository
2. The build system clones and compiles the coin core's consensus library
3. stratum-proxy links against the coin core's libraries
4. Block header hashing uses the coin core's native hash function:
   - `CBlockHeader::GetPoWHash()` for most altcoins (Litecoin, Rincoin, etc.)
   - `CBlockHeader::GetHash()` for Bitcoin and similar forks

This means any PoW algorithm changes pushed to the upstream coin repo are automatically
available after rebuilding.

## Documentation

- [BUILD.md](BUILD.md) - Detailed build instructions
- [COIN_CORES.md](COIN_CORES.md) - Ready-to-use configurations for various coins
- [README_AI.md](README_AI.md) - Technical documentation and architecture

## Tested Coins

| Coin | Version | PoW Algorithm | Build System |
|------|---------|---------------|---------------|
| Bitcoin Core | v28.0 | SHA256d | CMake |
| Bitcoin II | main | SHA256d | CMake |
| Fixedcoin | v29 | SHA256d | CMake |
| Rincoin | v1.0.2 | RinHash | Autotools |
| Yenten | yenten-6 | YescryptR16 | Autotools |

## License

See [LICENSE](LICENSE) for details.

## Releases

There are none and there will be none. The project is aimed at developers who should be able to create their own binaries.

## Help with the development

Help this project by forking it, adding more coins, testing it, enhancing it, correcting it and creating pull requests.

You can use AI tools for that. I've created README_AI.md for that - a human readable file that can be also used for an AI model to understand the project structure and its coding principles.

Majority of the code was created using the GitHub Copilot with Claude Opus 4.5 model.

## Donations

I don't want any donations for myself. If you find this project helpful and want to express your gratitude, look at the [Rincoin project](https://www.rincoin.net), create a wallet, buy or mine some Rincoins and spread a word about it.

## BTC PoW Lab sample

A ready connection file is available at `pool_samples/btcpowlab.toml`. Build the proxy against Bitcoin Core, copy that file to `stratum.toml`, and point one test miner at the local proxy port. The miner username should contain the Bitcoin reward address and an optional worker label.

The upstream endpoint is standard Stratum V1 over TCP. Pool rules, live status, and setup details are published at https://btcpowlab-pool.com/start.

Power CM Software operates BTC PoW Lab and contributed this sample. Inclusion does not imply endorsement by this project. Mining outcomes are probabilistic, and a share does not guarantee a block, reward, or profit.
