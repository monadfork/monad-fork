<div align="center">
  <img height="170x" src="https://avatars.githubusercontent.com/u/200391059?v=4" />

  <h1>Molana</h1>

  <p>
    <strong>The Ultimate High-Performance EVM-Compatible Blockchain</strong>
  </p>

  <p>
    <em>Combining Monad's Parallel Execution with Solana's Lightning Speed</em>
  </p>

  <p>
    <a href="#documentation"><img alt="Documentation" src="https://img.shields.io/badge/docs-comprehensive-blueviolet" /></a>
    <a href="#security-audits"><img alt="Security" src="https://img.shields.io/badge/audited-pending-yellow" /></a>
    <a href="https://opensource.org/licenses/Apache-2.0"><img alt="License" src="https://img.shields.io/github/license/molana/molana?color=blueviolet" /></a>
  </p>
</div>

---

## Executive Summary

**Molana** represents the next evolution in blockchain technology—a groundbreaking fork that merges the best innovations from two of 2025's most promising Layer-1 blockchains: **Monad** and **Solana**. By combining Monad's revolutionary parallel EVM execution with Solana's battle-tested high-throughput architecture, Molana delivers unprecedented performance while maintaining full Ethereum compatibility.

Monad achieves 10,000 transactions per second with 0.8-second finality through its parallel execution engine and MonadBFT consensus, while Solana has demonstrated 16 months of continuous uptime with replay times consistently below 400ms. Molana synthesizes these technologies to create a blockchain that processes transactions faster than either parent chain while preserving developer-friendly EVM compatibility.

**Key Performance Metrics (Theoretical):**
- **Throughput**: 50,000+ TPS (5x Monad, ~15x Solana peak sustained)
- **Finality**: 350ms average (faster than both parents)
- **EVM Compatibility**: 100% bytecode-compatible
- **Consensus**: Hybrid MolanaBFT (MonadBFT + Tower BFT fusion)
- **Block Time**: 300ms
- **Validator Requirements**: Optimized hardware (lower than Monad, higher than base Solana)

---

## The Genesis of Molana

### Why Fork Both Chains?

Monad was designed to address Ethereum's scalability limitations while maintaining full EVM compatibility, using optimistic parallel execution to process multiple transactions simultaneously. Meanwhile, Solana implemented confidential transfers, direct mapping improvements, and continued to focus on network speed and reliability.

We recognized that neither chain alone could achieve the performance ceiling demanded by the next generation of decentralized applications:

1. **Monad's Strengths**: Full EVM compatibility, parallel execution, developer familiarity
2. **Monad's Limitations**: Hardware requirements, relatively new codebase (mainnet launched Nov 24, 2025)
3. **Solana's Strengths**: Proven reliability, Proof-of-History timestamp mechanism, massive ecosystem
4. **Solana's Limitations**: No native EVM support, Rust/anchor learning curve for Ethereum devs

**Molana's Solution**: Merge Monad's parallelization strategy with Solana's Proof-of-History to create a hybrid consensus that processes EVM transactions at Solana-level speeds.

---

## Technical Architecture

### Core Innovations

#### 1. MolanaBFT Consensus

MolanaBFT is a novel consensus mechanism that fuses Monad's MonadBFT consensus with Solana's Tower BFT and Proof-of-History.

**Key Components:**

```rust
// Simplified MolanaBFT Implementation
pub struct MolanaConsensus {
    /// Monad-style parallel execution engine
    parallel_executor: ParallelExecutor,
    
    /// Solana-style Proof-of-History clock
    poh_clock: ProofOfHistoryClock,
    
    /// Hybrid BFT voting mechanism
    bft_voter: MolanaBFTVoter,
    
    /// Cross-chain state manager
    state_manager: HybridStateManager,
}

impl MolanaConsensus {
    /// Process block with parallel execution + PoH timestamps
    pub async fn process_block(&mut self, block: Block) -> Result<BlockResult> {
        // Generate PoH timestamp (Solana-style)
        let poh_entry = self.poh_clock.hash_and_record()?;
        
        // Execute transactions in parallel (Monad-style)
        let execution_results = self.parallel_executor
            .execute_parallel(block.transactions, poh_entry.hash)
            .await?;
        
        // Finalize with BFT consensus
        let finalized = self.bft_voter
            .vote_and_finalize(block.header, execution_results)
            .await?;
        
        Ok(BlockResult {
            poh_entry,
            execution_results,
            finalized_state: finalized,
        })
    }
}
```

**Advantages:**
- **Verifiable Time**: PoH provides cryptographic timestamps without external time sources
- **Parallel Safety**: Monad's optimistic parallel execution prevents race conditions
- **Fast Finality**: Combined BFT mechanisms reach consensus in ~350ms
- **EVM Native**: All Ethereum tooling works out-of-the-box

#### 2. Hybrid State Database (MolanaDB)

MolanaDB combines Monad's custom MonadDB database system with Solana's AccountsDB architecture.

**Architecture:**

```rust
// MolanaDB: Hybrid state storage
use monad_db::MonadDB;
use solana_accounts_db::AccountsDB;

pub struct MolanaDB {
    /// Monad-style key-value store for EVM state
    evm_state: MonadDB,
    
    /// Solana-style accounts database for native operations
    accounts: AccountsDB,
    
    /// Cross-reference index for hybrid operations
    cross_index: CrossChainIndex,
}

impl MolanaDB {
    /// Read EVM account state
    pub fn get_evm_account(&self, address: H160) -> Option<EvmAccount> {
        self.evm_state.get(&address.into())
    }
    
    /// Read Solana account state
    pub fn get_solana_account(&self, pubkey: &Pubkey) -> Option<SolanaAccount> {
        self.accounts.load(pubkey)
    }
    
    /// Unified state root for both systems
    pub fn compute_state_root(&self) -> StateRoot {
        let evm_root = self.evm_state.merkle_root();
        let solana_root = self.accounts.calculate_accounts_hash();
        
        // Combine roots using cryptographic hash
        StateRoot::from_hybrid(evm_root, solana_root)
    }
}
```

**Benefits:**
- **Dual Native Support**: Run EVM contracts AND Solana programs natively
- **Optimized Access**: Fast reads for both state models
- **Unified Security**: Single state root proves integrity of both systems

#### 3. Parallel Transaction Execution

Molana implements Monad's optimistic parallel execution with Solana's transaction batching.

```cpp
// C++ implementation for maximum performance (Monad-style)
class ParallelExecutor {
private:
    ThreadPool worker_pool_;
    ConflictDetector conflict_detector_;
    PoHClock& poh_clock_;  // Solana-style timing
    
public:
    // Execute transactions in parallel with PoH ordering
    std::vector<ExecutionResult> execute_parallel(
        const std::vector<Transaction>& txs,
        const Hash& poh_hash
    ) {
        std::vector<std::future<ExecutionResult>> futures;
        
        // Group transactions by potential conflicts
        auto tx_groups = conflict_detector_.group_non_conflicting(txs);
        
        for (auto& group : tx_groups) {
            // Submit parallel execution tasks
            for (auto& tx : group) {
                futures.push_back(worker_pool_.submit([this, tx, poh_hash]() {
                    return this->execute_tx_with_poh(tx, poh_hash);
                }));
            }
            
            // Wait for group to finish before starting next
            // (ensures causal ordering)
            for (auto& future : futures) {
                future.wait();
            }
            futures.clear();
        }
        
        // Collect results
        std::vector<ExecutionResult> results;
        for (auto& future : futures) {
            results.push_back(future.get());
        }
        
        return results;
    }
    
    ExecutionResult execute_tx_with_poh(
        const Transaction& tx,
        const Hash& poh_hash
    ) {
        // Execute transaction in isolated environment
        auto vm_result = evm_.execute(tx);
        
        // Stamp with PoH hash for verifiable ordering
        vm_result.poh_timestamp = poh_hash;
        
        return vm_result;
    }
};
```

**Performance Characteristics:**
- **Parallel Speedup**: Up to 10x faster than sequential execution
- **Deterministic Ordering**: PoH ensures reproducible transaction ordering
- **Conflict-Free Groups**: Smart batching maximizes parallelism
- **EVM Accuracy**: 100% compatibility with Ethereum execution semantics

---

## Network Specifications

### Performance Targets

| Metric | Molana | Monad | Solana | Ethereum |
|--------|--------|-------|--------|----------|
| **TPS (Theoretical)** | 50,000+ | 10,000 | 65,000 | 15-30 |
| **TPS (Sustained)** | 15,000+ | ~5,000 | 3,500 | 15-30 |
| **Block Time** | 300ms | 400ms | 400ms | 12s |
| **Finality** | 350ms | 800ms | 400ms | 13min |
| **Gas Model** | EVM-compatible | EVM-compatible | Compute units | EVM |
| **Consensus** | MolanaBFT | MonadBFT | Tower BFT + PoH | PoS (Gasper) |
| **Validator Hardware** | Medium-High | High | Medium | Low-Medium |

### Token Economics

**MOLA Token**:
- **Total Supply**: 1,000,000,000 MOLA (1 billion)
- **Initial Circulating**: 150,000,000 MOLA (15%)
- **Consensus**: Proof-of-Stake (hybrid staking model)
- **Inflation**: 5% annual, decreasing 10% per year
- **Use Cases**:
  - Gas fees for transactions
  - Validator staking (minimum 100,000 MOLA)
  - Governance voting
  - Priority transaction fees

**Distribution**:
```
├── Public Sale (20%): 200,000,000 MOLA
├── Ecosystem Fund (25%): 250,000,000 MOLA
├── Development Team (15%): 150,000,000 MOLA (4-year vesting)
├── Foundation Treasury (15%): 150,000,000 MOLA
├── Strategic Partners (10%): 100,000,000 MOLA (2-year vesting)
├── Community Incentives (10%): 100,000,000 MOLA
└── Liquidity Mining (5%): 50,000,000 MOLA
```

---

## Running a Molana Node

### Hardware Requirements

**Validator Node (Full Consensus Participation):**
- CPU: 32+ cores (AMD EPYC or Intel Xeon)
- RAM: 256GB minimum, 512GB recommended
- Storage: 2TB NVMe SSD (high IOPS required)
- Network: 1Gbps+ symmetrical connection
- GPU: Optional, but recommended for signature verification

**RPC Node (Read-Only):**
- CPU: 16+ cores
- RAM: 128GB
- Storage: 1TB NVMe SSD
- Network: 500Mbps+ connection

### Installation

```bash
# Install Rust toolchain
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

# Install Solana CLI (for compatibility)
sh -c "$(curl -sSfL https://release.solana.com/v1.18.0/install)"

# Clone Molana repository
git clone https://github.com/molana-labs/molana.git
cd molana

# Build from source
cargo build --release

# The binary will be at: target/release/molana
```

### Configuration

Create a configuration file at `~/.molana/config.toml`:

```toml
[network]
# Mainnet, testnet, or devnet
cluster = "mainnet"

# Public RPC endpoint
rpc_bind = "0.0.0.0:8899"

# WebSocket endpoint
ws_bind = "0.0.0.0:8900"

[consensus]
# Validator identity keypair
identity_keypair = "/path/to/validator-keypair.json"

# Vote account keypair
vote_keypair = "/path/to/vote-keypair.json"

# Staking configuration
stake_amount = 100000  # MOLA

[performance]
# Number of CPU cores for parallel execution
execution_threads = 28

# Transaction batch size
batch_size = 4096

# Enable aggressive caching
cache_enabled = true
cache_size_gb = 64

[evm]
# EVM execution backend
backend = "monad"  # Options: monad, revm, evmone

# Gas limit per block
block_gas_limit = 30000000

# Enable EVM tracing (performance impact)
tracing_enabled = false
```

### Starting the Node

```bash
# Validator node
./target/release/molana \
  --config ~/.molana/config.toml \
  --ledger /mnt/molana-ledger \
  --log /var/log/molana/validator.log

# RPC node (non-voting)
./target/release/molana \
  --config ~/.molana/config.toml \
  --rpc-only \
  --ledger /mnt/molana-ledger
```

---

## Developer Guide

### Deploying Smart Contracts

Molana is **100% EVM-compatible**, meaning any Ethereum smart contract can be deployed without modification.

**Using Hardhat:**

```javascript
// hardhat.config.js
module.exports = {
  networks: {
    molana: {
      url: "https://rpc.molana.network",
      chainId: 9999,  // Molana mainnet chain ID
      accounts: [process.env.PRIVATE_KEY],
      gasPrice: 1000000,  // 0.001 MOLA per gas unit
    },
    molanaTestnet: {
      url: "https://testnet-rpc.molana.network",
      chainId: 9998,
      accounts: [process.env.PRIVATE_KEY],
    },
  },
  solidity: {
    version: "0.8.24",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },
};
```

**Deploying a Contract:**

```bash
# Install dependencies
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox

# Compile contracts
npx hardhat compile

# Deploy to Molana mainnet
npx hardhat run scripts/deploy.js --network molana
```

**Example Contract:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/**
 * @title MolanaGreeter
 * @dev Example contract demonstrating EVM compatibility
 */
contract MolanaGreeter {
    string public greeting;
    address public owner;
    
    event GreetingChanged(string oldGreeting, string newGreeting);
    
    constructor(string memory _greeting) {
        greeting = _greeting;
        owner = msg.sender;
    }
    
    function setGreeting(string memory _greeting) external {
        require(msg.sender == owner, "Only owner can set greeting");
        
        string memory oldGreeting = greeting;
        greeting = _greeting;
        
        emit GreetingChanged(oldGreeting, _greeting);
    }
    
    function greet() external view returns (string memory) {
        return greeting;
    }
}
```

### Using Web3 Libraries

**Ethers.js:**

```typescript
import { ethers } from "ethers";

// Connect to Molana network
const provider = new ethers.JsonRpcProvider("https://rpc.molana.network");

// Create wallet
const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!, provider);

// Get account balance
const balance = await provider.getBalance(wallet.address);
console.log(`Balance: ${ethers.formatEther(balance)} MOLA`);

// Send transaction
const tx = await wallet.sendTransaction({
  to: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  value: ethers.parseEther("1.0"),
  gasLimit: 21000,
});

console.log(`Transaction hash: ${tx.hash}`);
await tx.wait();
console.log("Transaction confirmed!");
```

**Web3.js:**

```javascript
const Web3 = require("web3");

// Initialize Web3 with Molana RPC
const web3 = new Web3("https://rpc.molana.network");

// Load contract
const contract = new web3.eth.Contract(
  abi,
  "0x1234567890123456789012345678901234567890"
);

// Call contract method
const result = await contract.methods.greet().call();
console.log(`Greeting: ${result}`);

// Send transaction
await contract.methods
  .setGreeting("Hello, Molana!")
  .send({ from: accounts[0], gas: 100000 });
```

### Native Solana Program Calls (Advanced)

While Molana is primarily EVM-focused, you can also interact with native Solana programs for specific use cases:

```typescript
import { Connection, PublicKey, Transaction } from "@solana/web3.js";
import { Program, AnchorProvider } from "@coral-xyz/anchor";

// Connect to Molana (Solana-compatible endpoint)
const connection = new Connection("https://solana-rpc.molana.network");

// Call a native Solana program
const programId = new PublicKey("MoLaNa11111111111111111111111111111111111");
const program = new Program(idl, programId, provider);

const tx = await program.methods
  .yourInstruction()
  .accounts({
    // account configuration
  })
  .rpc();

console.log("Solana program transaction:", tx);
```

---

## Bridge Infrastructure

### Cross-Chain Compatibility

Molana maintains compatibility with both Ethereum and Solana ecosystems through native bridges.

**Supported Bridges:**
1. **Molana ↔ Ethereum**: Lock-and-mint bridge for ERC-20 tokens
2. **Molana ↔ Solana**: Native account mapping for SPL tokens
3. **Molana ↔ Bitcoin**: HTLC-based BTC bridge (via Lightning Network)
4. **Molana ↔ Monad**: Direct migration path for Monad applications

### Example: Bridging USDC from Ethereum

```typescript
import { ethers } from "ethers";

// Molana Bridge Contract on Ethereum
const BRIDGE_ADDRESS = "0xMoLaNaBrIdGe111111111111111111111111111";
const USDC_ADDRESS = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48";

// Connect to Ethereum
const ethProvider = new ethers.JsonRpcProvider(process.env.ETH_RPC_URL);
const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, ethProvider);

// Approve USDC
const usdc = new ethers.Contract(USDC_ADDRESS, usdcAbi, wallet);
await usdc.approve(BRIDGE_ADDRESS, ethers.parseUnits("100", 6));

// Bridge to Molana
const bridge = new ethers.Contract(BRIDGE_ADDRESS, bridgeAbi, wallet);
await bridge.bridgeToMolana(
  USDC_ADDRESS,
  ethers.parseUnits("100", 6),
  molanaRecipientAddress,
  { gasLimit: 200000 }
);

console.log("USDC bridged to Molana!");
```

---

## Ecosystem & Use Cases

### DeFi Applications

Molana's high throughput and low latency make it ideal for:

- **High-Frequency DEXs**: Order book-based exchanges with sub-second settlement
- **Perpetual Futures**: Leveraged trading with minimal liquidation risk
- **Automated Market Makers**: Lower slippage through higher liquidity depth
- **Lending Protocols**: Efficient liquidation mechanisms
- **Cross-Chain Yield Aggregators**: Bridge multiple L1s for optimal returns

### Gaming & NFTs

- **On-Chain Games**: Real-time gameplay with 350ms confirmation
- **Dynamic NFTs**: NFT metadata that updates based on game state
- **NFT Marketplaces**: Gas-efficient minting and trading
- **Play-to-Earn**: Microtransactions without prohibitive fees

### Enterprise & Institutional

- **Tokenized Securities**: Regulatory-compliant asset issuance
- **Supply Chain Tracking**: Real-time provenance verification
- **Central Bank Digital Currencies (CBDCs)**: Scalable payment rails
- **Decentralized Identity**: Privacy-preserving credential systems

---

## Governance

Molana utilizes an on-chain governance system inspired by both Compound and Solana's governance model.

### Governance Token

**MOL-GOV** (Governance MOLA):
- Obtained by staking MOLA tokens (1:1 ratio)
- Used for voting on protocol upgrades
- Non-transferable (locked during staking period)

### Proposal Process

1. **Discussion Phase** (7 days): Community discussion on forum
2. **Submission** (On-chain): Minimum 1M MOL-GOV to submit proposal
3. **Voting Period** (5 days): Token-weighted voting
4. **Execution Delay** (2 days): Time-lock before implementation
5. **Implementation**: Automated execution via governance contracts

### Example Governance Actions

- Protocol parameter changes (gas limits, block time, etc.)
- Treasury fund allocation
- Validator slashing conditions
- Bridge security threshold updates
- Emergency shutdown procedures

---

## Security

### Audits (Planned)

- **Trail of Bits**: Core consensus mechanism audit (Q1 2026)
- **Quantstamp**: Smart contract security audit (Q1 2026)
- **Certik**: Full-stack security audit (Q2 2026)
- **Hacken**: Bridge infrastructure audit (Q2 2026)

### Bug Bounty Program

Molana offers rewards for responsible disclosure of security vulnerabilities:

- **Critical**: Up to $500,000 MOLA
- **High**: Up to $100,000 MOLA
- **Medium**: Up to $25,000 MOLA
- **Low**: Up to $5,000 MOLA

Report vulnerabilities to: security@molana.network

### Known Limitations

As a newly forked blockchain, Molana inherits certain considerations:

1. **Mainnet Maturity**: Not yet battle-tested like parent chains
2. **Validator Decentralization**: Early-stage validator set
3. **EVM Parity**: Minor edge cases may differ from Ethereum mainnet
4. **Hardware Requirements**: Higher than typical PoS chains

---

## Roadmap

### 2025 Q4 ✅
- [x] Initial codebase fork from Monad and Solana
- [x] MolanaBFT consensus prototype
- [x] Testnet v1 launch
- [x] Developer documentation

### 2026 Q1 (Current)
- [ ] Mainnet beta launch
- [ ] Bridge deployment (Ethereum, Solana)
- [ ] Validator onboarding program
- [ ] Security audits (Trail of Bits, Quantstamp)
- [ ] Block explorer and analytics

### 2026 Q2
- [ ] DEX partnerships (Uniswap, Jupiter forks)
- [ ] Lending protocol integrations
- [ ] Mobile wallet support (MetaMask, Phantom)
- [ ] Institutional custody solutions

### 2026 Q3
- [ ] Layer-2 scaling solutions (zkEVM rollups)
- [ ] Privacy features (zk-SNARKs integration)
- [ ] Cross-chain messaging protocol
- [ ] DAO governance launch

### 2026 Q4
- [ ] Molana Improvement Proposals (MIPs) system
- [ ] Decentralized validator selection
- [ ] Advanced DeFi primitives (options, synthetics)
- [ ] Enterprise blockchain-as-a-service

---

## Community & Support

- **Website**: https://molana.network (placeholder)
- **Documentation**: https://docs.molana.network
- **GitHub**: https://github.com/molana-labs
- **Discord**: https://discord.gg/molana
- **Twitter**: https://twitter.com/molana_network
- **Telegram**: https://t.me/molana_official
- **Forum**: https://forum.molana.network

---

## FAQ

**Q: Is Molana a Layer-1 or Layer-2?**
A: Molana is a Layer-1 blockchain—it does not rely on Ethereum or Solana for security.

**Q: Can I deploy existing Ethereum contracts?**
A: Yes! Molana is 100% EVM bytecode-compatible. Simply change your RPC endpoint.

**Q: What's the difference between Molana and Monad?**
A: Molana integrates Solana's Proof-of-History and architectural optimizations to achieve higher throughput.

**Q: Can I use Solana wallets?**
A: For native Solana operations, yes. For EVM contracts, use Ethereum wallets (MetaMask, etc.).

**Q: When will mainnet launch?**
A: Mainnet beta is planned for Q1 2026 pending successful testnet completion and audits.

**Q: How do I become a validator?**
A: Stake a minimum of 100,000 MOLA and meet hardware requirements. See validator guide.

**Q: Are there transaction fees?**
A: Yes, similar to Ethereum gas fees. Typical transaction costs ~$0.0001-0.001.

**Q: Is Molana open source?**
A: Yes, fully open source under Apache 2.0 license.

---

## Technical Comparisons

### Molana vs. Monad

| Feature | Molana | Monad |
|---------|--------|-------|
| EVM Compatibility | ✅ Full | ✅ Full |
| Parallel Execution | ✅ Enhanced | ✅ Yes |
| Consensus | MolanaBFT (hybrid) | MonadBFT |
| Block Time | 300ms | 400ms |
| Finality | 350ms | 800ms |
| Throughput Target | 50,000 TPS | 10,000 TPS |
| Timestamp Mechanism | PoH + MonadBFT | MonadBFT only |
| Mainnet Status | Q1 2026 | Launched Nov 2025 |

### Molana vs. Solana

| Feature | Molana | Solana |
|---------|--------|--------|
| EVM Support | ✅ Native | ❌ No (Neon EVM exists) |
| Programming Model | Solidity + Rust | Rust (Anchor) |
| Consensus | MolanaBFT | Tower BFT + PoH |
| Block Time | 300ms | 400ms |
| Finality | 350ms | 400ms |
| Throughput (Theoretical) | 50,000 TPS | 65,000 TPS |
| Uptime (2025) | TBD | 16 months continuous |
| Tooling | Ethereum + Solana | Solana-specific |

---

## Contributing

We welcome contributions from the community! See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

**Ways to Contribute:**
- Report bugs via GitHub Issues
- Submit pull requests for bug fixes or features
- Improve documentation
- Participate in governance discussions
- Run testnet validators
- Build applications on Molana

---

## Disclaimer

**⚠️ EXPERIMENTAL SOFTWARE**

Molana is an experimental blockchain protocol combining two cutting-edge technologies. While we've carefully designed the system, unforeseen risks exist:

- **Smart Contract Risk**: Bugs in bridge contracts could result in loss of funds
- **Consensus Risk**: Novel consensus mechanisms may have edge cases
- **Network Risk**: Early-stage networks may experience downtime
- **Validator Risk**: Slashing conditions may result in stake loss

**Never invest more than you can afford to lose.** This software is provided "as-is" without warranty of any kind.

Cryptocurrency investments carry substantial risk. The information provided is for educational purposes and does not constitute financial advice.

---

## License

Molana is licensed under [Apache 2.0](./LICENSE).

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in Molana by you shall be licensed as above, without any additional terms or conditions.

---

## Acknowledgments

Molana stands on the shoulders of giants. We extend our gratitude to:

- **Monad Labs**: For pioneering parallel EVM execution
- **Solana Labs**: For proving high-performance blockchains are possible
- **Ethereum Foundation**: For creating the EVM standard
- **Anza**: For continued Solana development
- **Paradigm, Coinbase Ventures**: For funding blockchain innovation
- **The Open Source Community**: For making all of this possible

---

<div align="center">
  <p><strong>Molana: Where Monad Meets Solana</strong></p>
  <p><em>Parallel Execution. Proof-of-History. Unlimited Possibilities.</em></p>
  <br>
  <p>Building the fastest EVM-compatible blockchain in existence.</p>
</div>
