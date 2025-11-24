<div align="center">

![Monad Fork Banner](<img width="1728" height="595" alt="image" src="https://github.com/user-attachments/assets/5c33fa6e-98ef-4afe-b728-85976f980eeb" />)

# Monad Fork

**The People's Parallel EVM.**
<br>
10,000 TPS. Sub-second Finality. 0% VC Allocation.

[![License: MIT](https://img.shields.io/badge/License-MIT-purple.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://img.shields.io/badge/build-passing-success.svg)]()
[![Twitter](https://img.shields.io/badge/Twitter-Follow-blue.svg)](https://x.com/monadfork)
[![Discord](https://img.shields.io/badge/Discord-Join-7289DA.svg)]()

[Manifesto](#manifesto) • [Architecture](#architecture) • [Getting Started](#getting-started) • [Contributing](#contributing)

</div>

---

## ⚡️ Manifesto

**Monad Fork** is a high-performance, Ethereum-compatible Layer 1 blockchain.

We believe that high-throughput parallel execution technology should not be gatekept by venture capital or private access lists. This repository represents a fully open-source, community-governed implementation of parallel EVM execution, pipelined consensus, and asynchronous I/O.

**We didn't just fork the code. We forked the philosophy.**

## 🚀 Key Features

- **Parallel Execution:** Optimistic execution of transactions allows for massive throughput (10,000+ TPS) compared to sequential processing.
- **MonadBFT:** A high-performance consensus mechanism achieving sub-second finality.
- **Deferred Execution:** Consensus and execution are decoupled to maximize the block budget.
- **MonadDb:** A custom database optimized for storing blockchain state with asynchronous I/O, removing the bottlenecks of LevelDB/RocksDB.
- **Fully EVM Compatible:** Deploy existing Solidity contracts without changes. RPC compatible with MetaMask, Hardhat, and Foundry.

## 🛠 Architecture

### The Pipeline
Monad Fork utilizes a pipelined architecture to maximize efficiency:

1.  **Packet Processing:** Incoming transactions are filtered and propagated.
2.  **Consensus:** Nodes agree on transaction ordering (not execution).
3.  **Execution:** Transactions are executed in parallel based on the agreed ordering.
4.  **State Commitment:** Results are Merkleized and committed to MonadDb.

### Parallelism
Unlike traditional EVMs that process Txs one by one (`Tx1 -> Tx2 -> Tx3`), Monad Fork processes them simultaneously and resolves dependencies at the state level using optimistic concurrency control.

## 💻 Getting Started

### Prerequisites
- **Go**: v1.20+
- **Rust**: v1.70+ (for MonadDb backend)
- **Make**

### Building from Source

```bash
# Clone the repository
git clone https://github.com/monadfork/monadfork.git

# Enter directory
cd monadfork

# Install dependencies
make deps

# Build the node
make build
