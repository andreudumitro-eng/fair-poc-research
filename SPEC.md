F-PoC Research Specification
Fair Proof-of-Contribution: Technical Specification
Zcash-Oriented Edition v1.1
Research Lead: Andrii Dumitro
Version: 1.1 (Zcash Integration Ready)
Date: March 2026
License: MIT / Apache-2.0

Table of Contents
Introduction

Research Context & Relationship with Zcash

Equihash Integration Blueprint

Cryptographic Parameters

Block Structure

Shares and Proof-of-Contribution

Miner Identity

Bond (Economic Commitment)

Loyalty (Long-term Participation)

PoCI (Proof-of-Contribution Index)

Share Synchronization & Storage

Difficulty Adjustment

Transactions

Epoch Lifecycle

Security Analysis

Migration Path for Zcash Miners

Implementation Notes

Research Roadmap

Appendices

1. Introduction
This document provides the complete technical specification for the F-PoC (Fair Proof-of-Contribution) Research Prototype — a novel consensus mechanism designed to address structural weaknesses in ASIC-resistant Proof-of-Work networks, with specific focus on Zcash.

1.1 Core Principles
F-PoC redefines reward distribution in PoW networks. Instead of rewarding only the miner who finds a block, F-PoC distributes rewards across three independent dimensions of contribution:

Dimension	Weight	Description
Shares	40%	Computational work (valid hashes/solutions)
Loyalty	30%	Long-term participation consistency
Bond	30%	Economic commitment via locked collateral
*Note: Weights are optimized for Equihash (Zcash). The original 60/20/20 split was designed for Argon2id where ASIC/CPU ratio is lower.*

1.2 Key Research Innovation
Peaceful coexistence of ASIC, GPU, and CPU miners — F-PoC enables all miner types to participate profitably in the same network. ASICs retain their efficiency advantage, but smaller miners receive regular, predictable rewards, eliminating the "lottery problem" that currently excludes CPU/GPU miners from Zcash.

2. Research Context & Relationship with Zcash
2.1 What This Prototype Is
This is an open research platform for studying alternative reward distribution models in ASIC-resistant PoW networks. The codebase is a working implementation of F-PoC, designed to enable:

Simulation and analysis of reward variance reduction

Testing of loyalty and bonding mechanisms

Evaluation of ASIC/CPU coexistence conditions

Benchmarking of Equihash performance

2.2 What This Research Delivers to Zcash
Deliverable	Value to Zcash
Working F-PoC implementation	Ready-to-study codebase
Variance reduction analysis	Data on reward predictability
Loyalty mechanism evaluation	Understanding of long-term incentives
Bond/slashing security model	Economic alignment framework
Equihash integration blueprint	Clear path for implementation
ZIP-ready specification	Foundation for Zcash Improvement Proposal
3. Equihash Integration Blueprint
3.1 Why Equihash (Not Argon2id)
Zcash mainnet uses Equihash (n=200, k=9) . This research uses Argon2id as a placeholder during development, but the final integration must target Equihash.

Aspect	Research Prototype	Zcash Mainnet
PoW Function	Argon2id (256 MB)	Equihash (n=200, k=9)
Memory for Solving	256 MB	~2 GB
Memory for Verification	256 MB	~256 MB
Purpose	Rapid prototyping	Production consensus
3.2 Converting Equihash Solutions to Shares
Equihash generates solutions (not hashes). Each solution must be converted to a verifiable share:

text
// Equihash solution structure (for n=200, k=9)
solution = [index_0, index_1, ..., index_17]  // 18 indices, each 32 bits
solution_bytes = serialize(indices)            // 72 bytes total

// Convert to hash for difficulty comparison
share_hash = SHA256(solution_bytes)

// Validation
if share_hash < target_share → valid share
if share_hash < target_block → block found
3.3 Dynamic Share Difficulty (Anti-Spam)
Problem: With fixed target_share, large miners generate millions of shares per second, overwhelming the network.

Solution: Individual share difficulty based on bond size:

text
// Individual target for each miner
target_share_miner = target_block × SHARE_MULTIPLIER(miner)

where:
SHARE_MULTIPLIER(miner) = BASE_MULTIPLIER / (1 + log2(bond_miner + 1))
BASE_MULTIPLIER = 256
MIN_MULTIPLIER = 16  // cap for very large bonds
Effect:

Bond (ZEC)	log2(bond+1)	Multiplier	Share Difficulty
1	~1	128	Low (many shares)
100	~6.7	33	Medium
10,000	~13.3	17	High
1,000,000	~20	12	Very High
Result:

Small miners (low bond) generate more shares — compensating for low hash rate

Large ASIC farms (high bond) generate fewer shares per hash — reducing network load

3.4 Equihash Performance Benchmarks
Device	Equihash (200,9)	Shares/sec (estimated)	Normalized (√Shares)
CPU (Ryzen 7)	0.15 sol/s	~2	1.4
GPU (RTX 4090)	8 sol/s	~100	10
ASIC (Z17 Pro)	420 ksol/s	~5,000,000	2,236
Raw advantage: ASIC vs CPU = 2.5 million ×
After √ normalization: 2,236×
After dynamic difficulty (large bond penalty): ~100-200×
After weight adjustment (40% shares weight): ~40-80× effective advantage

Conclusion: CPU miners remain viable with consistent daily rewards, even if ASICs earn more.

4. Cryptographic Parameters
4.1 Zcash-Specific Parameters
rust
// ========== Zcash Mainnet Compatible ==========

// Time parameters (Zcash: 75 sec blocks)
pub const TARGET_BLOCK_TIME: u64 = 75;
pub const EPOCH_BLOCKS: u64 = 1152;      // 24 hours (1152 × 75 = 86,400 sec)
pub const EPOCH_DURATION: u64 = 86400;    // 24 hours in seconds

// Equihash (Zcash mainnet)
pub const EQUIHASH_N: u32 = 200;
pub const EQUIHASH_K: u32 = 9;
pub const EQUIHASH_SOLUTION_SIZE: usize = 1344;  // bytes for n=200,k=9

// PoCI weights (optimized for Equihash)
pub const POCI_WEIGHT_SHARES: f64 = 0.40;
pub const POCI_WEIGHT_LOYALTY: f64 = 0.30;
pub const POCI_WEIGHT_BOND: f64 = 0.30;

// Share parameters
pub const BASE_SHARE_MULTIPLIER: u64 = 256;
pub const MIN_SHARE_MULTIPLIER: u64 = 16;
pub const MAX_SHARES_PER_MINER_PER_EPOCH: u64 = 5000;

// Bond parameters (in zatoshis, 1 ZEC = 100,000,000 zatoshis)
pub const MINIMUM_BOND: u64 = 100_000_000;    // 1 ZEC
pub const BOND_LOCKUP_BLOCKS: u64 = 20160;    // ~14 days (20160 × 75 sec = 14 days)

// Slashing parameters
pub const CENSORSHIP_WARNING_BLOCKS: u64 = 100;
pub const CENSORSHIP_PENALTY_FIRST: f64 = 0.50;   // 50% bond
pub const CENSORSHIP_PENALTY_REPEAT: f64 = 1.00;  // 100% bond

// Difficulty adjustment
pub const DIFFICULTY_ADJUSTMENT_INTERVAL: u64 = 120;  // blocks
pub const TARGET_ADJUSTMENT_TIME: u64 = 9000;         // 120 × 75 sec
pub const MAX_DIFFICULTY_CHANGE: f64 = 0.25;          // ±25% per adjustment
4.2 Argon2id Parameters (Research Phase Only)
rust
// These are used ONLY during research/development
// Final production will use Equihash (above)

pub const ARGON2_MEMORY: u32 = 268_435_456;      // 256 MiB
pub const ARGON2_ITERATIONS: u32 = 2;
pub const ARGON2_PARALLELISM: u32 = 4;
pub const ARGON2_VERSION: u32 = 0x13;
5. Block Structure
5.1 Block Header (Zcash Compatible Extension)
rust
#[repr(C)]
pub struct BlockHeader {
    pub version: u32,                    // 4 bytes
    pub prev_hash: [u8; 32],            // 32 bytes
    pub merkle_root: [u8; 32],          // 32 bytes
    pub timestamp: u64,                  // 8 bytes
    pub difficulty: [u8; 32],           // 32 bytes (compact target)
    pub nonce: u64,                      // 8 bytes
    pub epoch_index: u32,               // 4 bytes (F-PoC extension)
    pub share_merkle_root: [u8; 32],    // 32 bytes (F-PoC extension)
    // Total: 172 bytes (vs 140 bytes in Zcash)
}
5.2 Block Validity
text
1. hash = Equihash(header_bytes)  // or Argon2id during research
2. hash < target_block (meets network difficulty)
3. timestamp > median(last_11_blocks.timestamp)  // anti-time-warp
4. timestamp < now + 2 hours
5. merkle_root correct for all transactions
6. share_merkle_root correct for all shares in epoch
7. No double-spend
8. Block height = prev_height + 1
5.3 Share Merkle Root
The last block of each epoch must include a Merkle root of all valid shares from that epoch:

rust
// Last block of epoch (height % EPOCH_BLOCKS == EPOCH_BLOCKS - 1)
share_merkle_root = Merkle(all_valid_shares_in_epoch)
This ensures consensus on share distribution and enables recovery after node crashes.

6. Shares and Proof-of-Contribution
6.1 Share Validity (Equihash Version)
text
// For Equihash
solution = solve(header, nonce)  // 1344 bytes
hash = SHA256(solution)

Share is valid if: hash < target_share_miner(miner_id)

Where target_share_miner(miner_id) = target_block × SHARE_MULTIPLIER(miner_id)
6.2 Share Packet (Network Transmission)
rust
#[repr(C)]
pub struct SharePacket {
    pub miner_id: [u8; 20],           // 20 bytes (RIPEMD160 of pubkey)
    pub header: BlockHeader,           // 172 bytes
    pub nonce: u64,                    // 8 bytes (original nonce used)
    pub solution: Vec<u8>,             // 1344 bytes (Equihash solution)
    // Total: ~1544 bytes per share
}
6.3 Share Limits
rust
pub const MAX_SHARES_PER_MINER_PER_EPOCH: u64 = 5000;
Shares exceeding this limit are ignored. This prevents single-miner dominance and limits DoS attack surface.

6.4 Share Validation Pipeline (Optimized)
text
┌─────────────────────────────────────────────────────────────┐
│ 1. Receive share packet (~1544 bytes)                      │
│ ↓                                                           │
│ 2. Quick syntax validation                                  │
│    - Check miner_id format                                  │
│    - Check solution size (must be 1344 bytes)              │
│    - Reject if miner exceeds MAX_SHARES this epoch         │
│ ↓                                                           │
│ 3. SHA256 prefilter (cheap rejection)                      │
│    - Compute hash = SHA256(solution)                       │
│    - Compare with target_share_miner                       │
│    - Rejects ~99% of invalid shares                        │
│    - Cost: <1 microsecond                                  │
│ ↓                                                           │
│ 4. Full Equihash verification (for shares that pass)       │
│    - Verify solution is valid for header                   │
│    - Cost: ~5-10ms on modern CPU                           │
│ ↓                                                           │
│ 5. Store valid share in share pool                         │
└─────────────────────────────────────────────────────────────┘
7. Miner Identity
rust
pub fn derive_miner_id(public_key: &[u8; 33]) -> [u8; 20] {
    // Compressed secp256k1 public key (33 bytes)
    let sha256_hash = sha256(public_key);
    ripemd160(&sha256_hash)  // 20 bytes
}
Properties:

Compatible with Bitcoin/Zcash P2PKH addressing

Deterministic: one public key = one miner_id

Cannot be reversed (pre-image resistant)

8. Bond (Economic Commitment)
8.1 Bond Structure
rust
pub struct Bond {
    pub miner_id: [u8; 20],
    pub amount: u64,                    // in zatoshis
    pub lock_until: u64,                // block height
    pub auditor_pubkey: Option<[u8; 33]>,  // optional auditor
    pub viewing_key: Option<[u8; 32]>,     // for transaction viewing
}
8.2 Bond Creation
Bond is created through a special transaction output:

text
scriptPubKey = OP_BOND <miner_id> <lock_height>
Rules:

Cannot be spent until lock_height + BOND_LOCKUP_BLOCKS

Multiple bond outputs from same miner_id accumulate

Bond participates in PoCI only if amount >= MINIMUM_BOND

8.3 Bond Normalization
rust
fn norm_bond(bond_amount: u64, max_bond: u64) -> f64 {
    if bond_amount < MINIMUM_BOND {
        return 0.0;
    }
    // Square root normalization to prevent whale dominance
    (bond_amount as f64).sqrt() / (max_bond as f64).sqrt()
}
8.4 Auditable Bonds (Regulatory Compliance)
For miners who need to demonstrate compliance (e.g., to exchanges or tax authorities):

rust
pub struct AuditableBond {
    pub miner_id: [u8; 20],
    pub amount: u64,
    pub lock_until: u64,
    pub auditor_pubkey: [u8; 33],      // auditor's public key
    pub viewing_key: [u8; 32],         // for viewing bond transactions
}

// Auditor can view:
// - Bond creation and balance
// - Reward payouts
// - Slashing events
// Auditor CANNOT:
// - Spend the bond
// - Modify the bond
// - View miner's private keys
Benefits:

Compliant with MiCA and other regulatory frameworks

Exchanges can list ZEC with F-PoC without compliance risk

Miners can prove legitimate operations

8.5 Slashing Conditions
Violation	Detection	Penalty
Equivocation	Same miner signs two blocks at same height	100% bond burned
51% Attack Participation	Two competing blocks at same height from same miner	100% bond burned
Censorship (First)	Miner's blocks exclude certain transaction types for >100 blocks	50% bond burned
Censorship (Repeat)	Second offense within 30 days	100% bond burned
Invalid Share Flooding	Invalid shares >30% of total in an epoch	100% bond burned
Outdated Mining	Mining on old version after network upgrade	10% bond per block (max 100%)
8.6 Slashing Transaction
rust
// Special transaction type for slashing
pub struct SlashTransaction {
    pub bond_txid: [u8; 32],           // Bond being slashed
    pub proof: SlashProof,              // Evidence of violation
}

pub enum SlashProof {
    Equivocation {
        block1: BlockHeader,
        block2: BlockHeader,
    },
    Censorship {
        start_height: u64,
        end_height: u64,
        excluded_tx_types: Vec<TxType>,
    },
    // ... other proof types
}
9. Loyalty (Long-term Participation)
9.1 Loyalty Mechanism
rust
pub struct LoyaltyState {
    pub miner_id: [u8; 20],
    pub loyalty_score: u64,            // cumulative participation
    pub last_epoch: u64,               // last epoch with shares
    pub missed_epochs: u64,            // consecutive misses
}
9.2 Loyalty Update Rules
rust
fn update_loyalty(state: &mut LoyaltyState, participated: bool, current_epoch: u64) {
    if participated {
        // Participated this epoch
        if state.missed_epochs > 0 {
            // Returning after absence
            let grace_factor = if state.missed_epochs <= 3 { 0.5 } else { 0.7 };
            state.loyalty_score = (state.loyalty_score as f64 * grace_factor) as u64;
            state.missed_epochs = 0;
        } else {
            // Continuous participation
            state.loyalty_score += 1;
        }
    } else {
        // Missed this epoch
        state.missed_epochs += 1;
        if state.missed_epochs <= 3 {
            state.loyalty_score = (state.loyalty_score as f64 * 0.7) as u64;
        } else {
            state.loyalty_score /= 2;
        }
    }
    state.last_epoch = current_epoch;
}
9.3 Loyalty Normalization
rust
fn norm_loyalty(loyalty: u64, max_loyalty: u64) -> f64 {
    loyalty as f64 / max_loyalty as f64
}
9.4 Example Scenarios
Continuous Miner:

text
Epoch 1: loyalty = 1
Epoch 2: loyalty = 2
...
Epoch 100: loyalty = 100
Miner with 5-day outage:

text
Epoch 100: loyalty = 100
Epoch 101: missed → loyalty = 70
Epoch 102: missed → loyalty = 49
Epoch 103: missed → loyalty = 34
Epoch 104: returns → loyalty = 17 (grace period recovery)
Epoch 105: participates → loyalty = 18
10. PoCI (Proof-of-Contribution Index)
10.1 Complete Formula
rust
fn calculate_poci(
    shares: u64,
    loyalty: u64,
    bond: u64,
    max_shares: u64,
    max_loyalty: u64,
    max_bond: u64,
) -> f64 {
    // Square root normalization for shares and bond
    let norm_shares = (shares as f64).sqrt() / (max_shares as f64).sqrt();
    let norm_loyalty = loyalty as f64 / max_loyalty as f64;
    let norm_bond = (bond as f64).sqrt() / (max_bond as f64).sqrt();
    
    // Weighted sum
    0.40 * norm_shares + 0.30 * norm_loyalty + 0.30 * norm_bond
}
10.2 Reward Calculation
rust
fn calculate_rewards(
    miners: &[MinerState],
    epoch_reward: u64,
    total_fees: u64,
) -> Vec<u64> {
    let total_reward = epoch_reward + total_fees;
    let total_poci: f64 = miners.iter().map(|m| m.poci).sum();
    
    miners.iter().map(|m| {
        ((m.poci / total_poci) * total_reward as f64) as u64
    }).collect()
}
10.3 Complete Example
Epoch with 3 miners:

Miner	Shares	Loyalty	Bond (ZEC)	√Shares	Norm Shares	Norm Loyalty	Norm Bond	PoCI	Reward (%)
ASIC Farm	5,000	100	10,000	70.71	1.00	1.00	1.00	0.400 + 0.300 + 0.300 = 1.000	42.6%
GPU Rig	500	50	100	22.36	0.316	0.50	0.316	0.126 + 0.150 + 0.095 = 0.371	15.8%
CPU Miner	100	10	1	10.00	0.141	0.10	0.141	0.056 + 0.030 + 0.042 = 0.128	5.4%
Observation: All miners receive rewards every epoch. The CPU miner earns ~5% of epoch reward consistently, not waiting months for a lucky block.

11. Share Synchronization & Storage
11.1 Persistence Layer (RocksDB)
rust
pub struct ShareStorage {
    db: RocksDB,
    cache: LruCache<[u8; 32], ShareRecord>,  // Hot cache for recent shares
}

impl ShareStorage {
    pub fn add_share(&mut self, share: ShareRecord) -> Result<()> {
        // Add to hot cache
        self.cache.put(share.hash, share.clone());
        
        // If cache exceeds limit, flush oldest to disk
        if self.cache.len() > HOT_CACHE_LIMIT {
            let (_, oldest) = self.cache.pop_lru();
            self.db.put(encode_key(&oldest.hash), serialize(&oldest))?;
        }
        
        Ok(())
    }
    
    pub fn get_share(&self, hash: &[u8; 32]) -> Option<ShareRecord> {
        // Check cache first
        if let Some(share) = self.cache.get(hash) {
            return Some(share);
        }
        // Fall back to disk
        self.db.get(encode_key(hash)).ok().flatten().map(deserialize)
    }
}
11.2 Checkpoint System
rust
// Create checkpoint every 10 blocks
fn create_checkpoint(block_height: u64, share_pool: &SharePool) -> Result<()> {
    let epoch = block_height / EPOCH_BLOCKS;
    let checkpoint_key = format!("epoch_{}_height_{}", epoch, block_height);
    
    let checkpoint = Checkpoint {
        epoch,
        block_height,
        share_merkle_root: share_pool.merkle_root(),
        share_count: share_pool.len(),
        timestamp: SystemTime::now(),
    };
    
    db.put(&checkpoint_key, serialize(checkpoint))?;
    db.put(&format!("shares_{}", checkpoint_key), serialize(share_pool))?;
    
    // Keep last 10 checkpoints
    cleanup_old_checkpoints(10);
    
    Ok(())
}

// Recovery after crash
fn recover(block_height: u64) -> Result<SharePool> {
    let checkpoint = find_last_checkpoint(block_height)?;
    let mut share_pool: SharePool = deserialize(db.get(&format!("shares_{}", checkpoint.key))?);
    
    // Replay blocks from checkpoint height to current
    for height in checkpoint.block_height..block_height {
        let block = get_block(height)?;
        replay_block(&mut share_pool, &block)?;
    }
    
    Ok(share_pool)
}
11.3 Epoch Commit Consensus
The last block of each epoch contains the Merkle root of all valid shares:

rust
// In last block of epoch (height % EPOCH_BLOCKS == EPOCH_BLOCKS - 1)
block.share_merkle_root = share_pool.merkle_root();

// Nodes verify:
assert!(block.share_merkle_root == local_share_pool.merkle_root());
If a node has a different root:

Request missing shares from peers via getshares message

Majority vote from ≥5 peers determines correct root

Nodes with persistent mismatches (>3 times in 10 epochs) are banned

12. Difficulty Adjustment
12.1 Zcash-Compatible Adjustment
rust
fn adjust_difficulty(blocks: &[Block], current_target: [u8; 32]) -> [u8; 32] {
    let interval = DIFFICULTY_ADJUSTMENT_INTERVAL;  // 120 blocks
    let target_time = interval * TARGET_BLOCK_TIME; // 120 × 75 = 9000 sec
    
    let actual_time = blocks.last().timestamp - blocks.first().timestamp;
    
    // Convert target to integer for calculation
    let target_int = target_to_u256(current_target);
    let new_target = target_int * actual_time as u128 / target_time as u128;
    
    // Clamp to ±25%
    let max_target = target_int * (1 + MAX_DIFFICULTY_CHANGE as u128);
    let min_target = target_int * (1 - MAX_DIFFICULTY_CHANGE as u128);
    
    let clamped = new_target.clamp(min_target, max_target);
    u256_to_target(clamped)
}
12.2 Time-Warp Attack Protection
rust
fn validate_timestamp(block: &Block, last_11_blocks: &[Block]) -> bool {
    let median_timestamp = median(last_11_blocks.iter().map(|b| b.timestamp));
    block.timestamp > median_timestamp && block.timestamp < now() + 7200
}
13. Transactions
13.1 Transaction Structure (Zcash Compatible)
rust
pub struct Transaction {
    pub version: u32,
    pub inputs: Vec<TxIn>,
    pub outputs: Vec<TxOut>,
    pub locktime: u32,
    pub joinsplits: Vec<JoinSplit>,  // Zcash shielded transactions
    pub binding_sig: [u8; 64],        // For shielded transactions
}

pub struct TxIn {
    pub prev_txid: [u8; 32],
    pub prev_index: u32,
    pub script_sig: Vec<u8>,
    pub sequence: u32,
}

pub struct TxOut {
    pub value: u64,                    // in zatoshis
    pub script_pubkey: Vec<u8>,
}
13.2 Special Script Types
Script	Description
OP_BOND <miner_id> <lock_height>	Bond output (locked for BOND_LOCKUP_BLOCKS)
OP_SLASH_EQUIVOCATION <proof>	Slashing transaction
OP_REWARD_EPOCH <epoch>	Epoch reward distribution (coinbase-like)
13.3 Validation Rules
rust
fn validate_transaction(tx: &Transaction, utxo_set: &UTXOSet) -> Result<()> {
    // Balance check
    let inputs_sum: u64 = tx.inputs.iter()
        .map(|input| utxo_set.get_value(&input.prev_txid, input.prev_index))
        .sum();
    let outputs_sum: u64 = tx.outputs.iter().map(|out| out.value).sum();
    let fee = inputs_sum - outputs_sum;
    
    ensure!(fee >= MINIMUM_FEE, Error::InsufficientFee);
    
    // No double-spend
    for input in &tx.inputs {
        ensure!(!utxo_set.is_spent(input), Error::DoubleSpend);
    }
    
    // Signature validation
    for (input, utxo) in tx.inputs.iter().zip(utxos) {
        verify_signature(&input.script_sig, &utxo.script_pubkey, &tx.hash())?;
    }
    
    Ok(())
}
14. Epoch Lifecycle
14.1 Epoch Structure
text
Epoch N: blocks [N × EPOCH_BLOCKS, (N+1) × EPOCH_BLOCKS - 1]
- EPOCH_BLOCKS = 1152 (24 hours at 75 sec/block)
- Epoch 0: blocks 0-1151
- Epoch 1: blocks 1152-2303
- etc.
14.2 During Epoch
Miners:

Perform Equihash(header, nonce)

If hash < target_block → found block, broadcast

If hash < target_share_miner(miner_id) → valid share, broadcast

Nodes:

Validate all blocks and shares

Maintain share pool for current epoch (in memory + disk)

Track loyalty of each miner

Track bond of each miner

Create checkpoints every 10 blocks

14.3 At Epoch Boundary (Block N × 1152 - 1)
rust
fn finalize_epoch(epoch: u64, share_pool: &SharePool) -> Result<()> {
    // 1. Calculate PoCI for all miners
    let max_shares = share_pool.max_shares();
    let max_loyalty = loyalty_state.max_loyalty();
    let max_bond = bond_state.max_bond();
    
    let poci_scores: Vec<(MinerId, f64)> = share_pool.miners()
        .map(|miner| {
            let poci = calculate_poci(
                miner.shares, miner.loyalty, miner.bond,
                max_shares, max_loyalty, max_bond
            );
            (miner.id, poci)
        })
        .collect();
    
    // 2. Calculate rewards
    let total_reward = EPOCH_REWARD + total_fees_in_epoch(epoch);
    let total_poci: f64 = poci_scores.iter().map(|(_, p)| p).sum();
    
    let rewards: Vec<(MinerId, u64)> = poci_scores.iter()
        .map(|(id, poci)| (*id, ((poci / total_poci) * total_reward as f64) as u64))
        .collect();
    
    // 3. Generate payout transactions
    for (miner_id, reward) in rewards {
        create_reward_transaction(miner_id, reward, epoch);
    }
    
    // 4. Clear share pool for next epoch
    share_pool.clear();
    
    // 5. Update loyalty (decay for miners who missed epoch)
    loyalty_state.apply_decay(share_pool.active_miners());
    
    // 6. Adjust difficulty based on last 120 blocks
    adjust_difficulty();
    
    Ok(())
}
15. Security Analysis
15.1 ASIC Resistance
Mechanism	Effect
Equihash memory hardness	2 GB memory requirement for solving
Dynamic share difficulty	Large bonds reduce share rate
Square-root normalization	Limits advantage of high share counts
Conclusion: ASICs remain profitable but do not exclude CPU/GPU miners.

15.2 Sybil Resistance
Attack: Create 10,000 miner_ids to capture disproportionate rewards.

Defense:

Minimum bond of 1 ZEC per miner_id

Bond locked for 14 days

Cost of attack: 10,000 ZEC locked for 14 days (~$140,000 at current prices)

Conclusion: Sybil attacks are economically infeasible.

15.3 51% Attack Protection
Requirements for 51% attack:

51% of all epoch shares (computational control)

High loyalty scores (long-term participation)

Significant bond at risk (economic commitment)

Economic cost:

Bond at risk: at least 10,000+ ZEC

Lost loyalty: months of accumulated score

Slashing penalty: 100% bond burn

Conclusion: 51% attack in F-PoC is economically irrational.

15.4 Censorship Resistance
Detection: Nodes track transaction inclusion patterns per miner.

Penalty:

First offense: 50% bond burn

Repeat offense: 100% bond burn

Result: Miners have economic incentive to include all valid transactions.

15.5 DoS Protection
Attack Vector	Mitigation
Share flooding	MAX_SHARES_PER_MINER_PER_EPOCH
Invalid share flood	Bond slashing for >30% invalid
Memory exhaustion	LRU cache + disk persistence
Network spam	Rate limiting per IP
16. Migration Path for Zcash Miners
16.1 Transition Timeline
Phase	Duration	Description
Phase 1: Testnet	3 months	F-PoC running on Zcash testnet
Phase 2: Signaling	1 month	Miners signal readiness via coinbase
Phase 3: Activation	At block height X	F-PoC activates via network upgrade
Phase 4: Coexistence	2 weeks	Both old and new rules accepted
Phase 5: Finalization	After 2 weeks	Old PoW disabled
16.2 Miner Upgrade Steps
For Pool Operators:

Update stratum server to accept shares

Implement bond management (miners lock ZEC)

Update payout logic for epoch-based distribution

For Solo Miners:

Create bond transaction (min 1 ZEC)

Update mining software to F-PoC version

Monitor loyalty score for optimal uptime

16.3 Bond Migration
Existing ZEC holders can participate without mining:

rust
// Non-mining bond (earns only from loyalty/bond components)
pub struct StakingBond {
    pub amount: u64,
    pub lock_until: u64,
    // No mining capability, but earns from 60% of rewards
}
This allows ZEC holders to earn rewards without mining hardware.

17. Implementation Notes
17.1 Rust Implementation Status
Component	Status	Completion
Constants & Types	✅ Complete	100%
Error Handling	✅ Complete	100%
Blocks	✅ Complete	100%
Transactions	✅ Complete	100%
Equihash Integration	🔄 In Progress	40%
PoCI Calculation	✅ Complete	100%
Loyalty Tracking	✅ Complete	100%
Bond Management	🔄 In Progress	60%
Share Storage (RocksDB)	✅ Complete	100%
RPC API	✅ Complete	100%
P2P Network	🔄 In Progress	40%
Audit/Bond Viewing	🔄 In Progress	30%
17.2 Key Data Types
rust
// All monetary values in zatoshis (1 ZEC = 100,000,000 zatoshis)
pub type Amount = u64;

// Miner identity (20 bytes, RIPEMD160 of pubkey)
pub type MinerId = [u8; 20];

// Block hash (32 bytes)
pub type BlockHash = [u8; 32];

// Transaction hash (32 bytes)
pub type Txid = [u8; 32];
17.3 Error Handling
rust
#[derive(Debug, thiserror::Error)]
pub enum FPoCError {
    #[error("Invalid block: {0}")]
    InvalidBlock(String),
    
    #[error("Invalid share: {0}")]
    InvalidShare(String),
    
    #[error("Insufficient bond: need {need}, have {have}")]
    InsufficientBond { need: Amount, have: Amount },
    
    #[error("Bond not locked: unlocks at {unlock_height}")]
    BondNotLocked { unlock_height: u64 },
    
    #[error("Slashing condition met: {violation}")]
    Slashing { violation: String },
    
    #[error("Share limit exceeded: max {max}")]
    ShareLimitExceeded { max: u64 },
}
18. Research Roadmap
Phase	Period	Description	Status
Phase 1	Q1 2026	Theoretical analysis, mathematical modeling	✅ Completed
Phase 2	Q2 2026	Argon2id prototype (Rust)	✅ 70% Complete
Phase 3	Q3 2026	Simulation & benchmarking	⏳ Planned
Phase 4a	Q3 2026	Equihash integration (core)	🔴 In Progress
Phase 4b	Q4 2026	Zcash testnet deployment	⏳ Planned
Phase 5	Q1 2027	ZIP (Zcash Improvement Proposal)	⏳ Planned
Phase 6	Q2 2027	Mainnet activation (if approved)	⏳ TBD
19. Appendices
Appendix A: Complete PoCI Calculation Example
Scenario: Epoch with 3 miners using Equihash

Miner	Shares	Loyalty	Bond (ZEC)
ASIC Farm	5,000	100	10,000
GPU Rig	500	50	100
CPU Miner	100	10	1
Step 1: Square root normalization

text
max_shares = 5,000
√max = 70.71

norm_shares_A = √5000 / 70.71 = 70.71 / 70.71 = 1.00
norm_shares_B = √500 / 70.71 = 22.36 / 70.71 = 0.316
norm_shares_C = √100 / 70.71 = 10.00 / 70.71 = 0.141
Step 2: Loyalty normalization

text
max_loyalty = 100
norm_loyalty_A = 100/100 = 1.00
norm_loyalty_B = 50/100 = 0.50
norm_loyalty_C = 10/100 = 0.10
Step 3: Bond normalization

text
max_bond = 10,000 ZEC
√max = 100

norm_bond_A = √10000 / 100 = 100/100 = 1.00
norm_bond_B = √100 / 100 = 10/100 = 0.10
norm_bond_C = √1 / 100 = 1/100 = 0.01
Step 4: PoCI calculation

text
PoCI_A = 0.40×1.00 + 0.30×1.00 + 0.30×1.00 = 1.00
PoCI_B = 0.40×0.316 + 0.30×0.50 + 0.30×0.10 = 0.126 + 0.150 + 0.030 = 0.306
PoCI_C = 0.40×0.141 + 0.30×0.10 + 0.30×0.01 = 0.056 + 0.030 + 0.003 = 0.089
Step 5: Reward distribution (assuming 100 ZEC epoch reward)

text
ΣPoCI = 1.00 + 0.306 + 0.089 = 1.395

Reward_A = (1.00/1.395) × 100 = 71.7 ZEC
Reward_B = (0.306/1.395) × 100 = 21.9 ZEC
Reward_C = (0.089/1.395) × 100 = 6.4 ZEC
Appendix B: Variance Analysis
Traditional PoW (Zcash current):

Coefficient of variation (CV) > 1.0 for small miners

Expected waiting time for 0.01% miner: ~3-4 days with high variance

CPU miner may never find a block

F-PoC:

Coefficient of variation ≈ 0.1

All miners receive rewards every 24 hours

Predictable income stream for all participants

Appendix C: Performance Benchmarks (Equihash)
Device	Solutions/sec	Shares/sec	Est. Daily Reward (100 ZEC epoch)
CPU (Ryzen 7)	0.15	~2	6-8 ZEC
GPU (RTX 4090)	8	~100	20-25 ZEC
ASIC (Z17 Pro)	420,000	~5M	70-75 ZEC
Appendix D: Glossary
Term	Definition
Bond	Locked ZEC that qualifies a miner for PoCI rewards
Epoch	24-hour period (1152 blocks) for reward calculation
Loyalty	Score based on consecutive epochs with participation
PoCI	Proof-of-Contribution Index — composite score for rewards
Share	Valid Equihash solution meeting target_share_miner
Slashing	Penalty (bond burn) for malicious behavior
References
Zcash Protocol Specification, version 2024.1

Equihash: Memory-hard Proof-of-Work, Biryukov & Khovratovich

RFC 9106: Argon2 Memory-Hard Function

Zcash Improvement Proposals (ZIPs) 200, 201, 244

Document Version: 1.1 (Zcash-Oriented Edition)
Last Updated: March 2026
Research Lead: Andrii Dumitro
Contact: Available via GitHub
