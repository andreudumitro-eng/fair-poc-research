F-PoC Research Specification
Fair Proof-of-Contribution: Technical Specification for Research Prototype

Research Lead: Andrii Dumitro
Version: 1.0 (Research Prototype)
Date: March 2026

Table of Contents
Introduction

Cryptographic Parameters

Block Structure

Shares and Proof-of-Contribution

Miner Identity

Bond (Economic Commitment)

Loyalty (Long-term Participation)

PoCI (Proof-of-Contribution Index)

Share Synchronization Between Nodes

Difficulty Adjustment

Transactions

Epoch Lifecycle

Security Analysis

Implementation Notes

Appendices

Introduction
This document provides the complete technical specification for the F-PoC (Fair Proof-of-Contribution) Research Prototype — a novel consensus mechanism designed to address structural weaknesses in ASIC-resistant Proof-of-Work networks, with specific focus on Zcash.

Core Principles
F-PoC redefines reward distribution in PoW networks. Instead of rewarding only the miner who finds a block, F-PoC distributes rewards across three independent dimensions of contribution:

Dimension	Weight	Description
Shares	60%	Computational work (valid hashes)
Loyalty	20%	Long-term participation consistency
Bond	20%	Economic commitment via locked collateral
Key Research Innovation
Peaceful coexistence of ASIC, GPU, and CPU miners — F-PoC enables all miner types to participate profitably in the same network. ASICs retain their efficiency advantage, but smaller miners receive regular, predictable rewards, eliminating the "lottery problem" that currently excludes CPU/GPU miners from Zcash.

Cryptographic Parameters
Proof-of-Work Function: Argon2id
Parameter	Value	Rationale
Memory	256 MiB	ASIC development cost >$50,000 per chip
Iterations	2	Balance: ~100ms on modern CPU
Parallelism	4	Optimal for 4-8 core CPUs
Version	0x13	Argon2id (hybrid)
Hash output	256 bits	Full entropy for difficulty
Why Argon2id for Research?
This research uses Argon2id as a proxy for memory-hard PoW functions. Future phases will adapt F-PoC to work with Zcash's Equihash.

Performance Benchmarks
Hardware	Time/Hash	Throughput	Relative to CPU
CPU (Ryzen 7 5700X)	~110 ms	9.1 H/s	1.0×
CPU (Intel i7-12700K)	~95 ms	10.5 H/s	1.15×
GPU (RTX 3090)	~45 ms	22 H/s	2.4×
Hypothetical Argon2 ASIC	~5 ms	200 H/s	~20×
Key Insight: Even with hypothetical ASICs, the advantage is limited to ~20× (compared to 1000×+ for SHA256). This makes peaceful coexistence feasible.

Block Structure
Block Header (120 bytes, little-endian)
Field	Type	Size	Description
version	uint32	4	Block version (current: 1)
prev_hash	[32]byte	32	Hash of previous block header
merkle_root	[32]byte	32	Root of transaction Merkle tree
timestamp	uint64	8	Unix timestamp
difficulty	[32]byte	32	Compact target representation
nonce	uint64	8	Nonce for Proof-of-Work
epoch_index	uint32	4	Current epoch number (starting at 1)
Block Validity
text
hash = Argon2id(header_bytes)
Block is valid if:

hash < target_block (meets network difficulty)

timestamp > median(last_11_blocks.timestamp)

timestamp < now + 2 hours (protection against time-warp attacks)

merkle_root correct for all transactions

No double-spend

Block height = prev_height + 1

Shares and Proof-of-Contribution
Share Validity
text
hash = Argon2id(header_bytes)
Share is valid if: hash < target_share

Protocol rule: target_share = target_block << 8

This means each found block corresponds to ~256 valid shares, ensuring low reward variance and making solo mining viable.

Share Packet (180 bytes)
Field	Type	Size	Description
miner_id	[20]byte	20	Miner identity (RIPEMD160 of pubkey)
header	[120]byte	120	Complete block header
nonce	uint64	8	Nonce that produced valid share
hash	[32]byte	32	Computed Argon2id hash
Share Limits
text
MAX_SHARES_PER_MINER_PER_EPOCH = 5000
Shares exceeding this limit are ignored (prevents single-miner dominance).

Share Validation Pipeline
text
┌─────────────────────────────────────────────────────────────┐
│ 1. Receive share packet (180 bytes)                        │
│ ↓                                                           │
│ 2. SHA256 prefilter (cheap rejection)                      │
│    - Cost: <1 microsecond                                  │
│    - Rejects ~90% of invalid shares                        │
│ ↓                                                           │
│ 3. Cache lookup (60-second TTL)                            │
│    - Cache size: 10,000 entries                            │
│    - Hit rate: 70-80%                                      │
│ ↓                                                           │
│ 4. Argon2id computation                                     │
│    - Cost: ~100ms on modern CPU                            │
│ ↓                                                           │
│ 5. Store result in cache                                   │
└─────────────────────────────────────────────────────────────┘
Invalid Share Monitoring
Nodes track invalid_shares / total_shares per miner_id:

Threshold	Action
>10%	Warning logged
>30%	Temporary mining ban (3 epochs)
Miner Identity
text
miner_id = RIPEMD160(SHA256(compressed_secp256k1_pubkey))
Properties:

Compatible with Bitcoin P2PKH addressing

Deterministic: one public key = one miner_id

Cryptographically secure, cannot be reversed

All PoCI accruals and rewards are strictly tied to miner_id.

Bond (Economic Commitment)
Parameters
Parameter	Value	Rationale
Minimum for PoCI	1 unit	Economic barrier against Sybil attacks
Lock-up period	20,160 blocks	~14 days (prevents gaming)
Bond weight in PoCI	20%	Economic alignment
Bond Mechanism
Bond is created through a special output: scriptPubKey = OP_BOND

Properties:

Cannot be spent until lock_height + 20,160

Multiple bond outputs from same miner_id accumulate

Bond participates in PoCI only if >= MINIMUM_BOND

Bond Normalization
If bond_i >= MINIMUM_BOND:

text
norm_bond_i = bond_i / max_bond_in_epoch
If bond_i < MINIMUM_BOND:

text
norm_bond_i = 0
Slashing (100% Bond Burn)
Bond is burned in cases of:

Violation	Detection	Penalty
Equivocation	Same miner signs two blocks at same height	100% bond burned
51% Attack Participation	Proven via network analysis (future phase)	100% bond burned
Equivocation Slashing Process:

Node detects two blocks at height H from same miner_id

Creates SLASH_EQUIVOCATION transaction with both headers

Broadcasts to network

When included in block: all bond of this miner_id is burned

Loyalty (Long-term Participation)
Mechanism
text
Initial: loyalty = 0

Epoch participation (≥1 valid share):
    loyalty = loyalty + 1

Missed epoch (no valid shares):
    loyalty = max(loyalty * 0.7, loyalty // 2)
Grace period: First 3 missed epochs after returning use decay factor 0.5 (faster recovery).

Examples
Scenario A: Continuous Participation

text
Epoch 1: 0 → 1
Epoch 2: 1 → 2
Epoch 3: 2 → 3
Epoch 100: 99 → 100
Scenario B: Long Break

text
Epoch 1-100: loyalty = 100
Epoch 101: 100 → 70 (missed, *0.7)
Epoch 102: 70 → 49 (missed, *0.7)
Epoch 103: 49 → 34 (missed, *0.7)
Epoch 104: 34 → 17 (missed, //2)
Epoch 105: 17 → 8 (missed, //2)
Loyalty Normalization
text
norm_loyalty_i = loyalty_i / max_loyalty_in_epoch
PoCI (Proof-of-Contribution Index)
Basic Formula
text
PoCI_i = 0.6 × norm_shares_i + 0.2 × norm_loyalty_i + 0.2 × norm_bond_i
Component Normalization
Each component is normalized to [0, 1] range:

text
norm_X_i = X_i / max_X_in_epoch
Shares Normalization (Square Root)
To prevent dominant miners from monopolizing the shares component:

text
norm_shares_i = sqrt(shares_raw_i) / max_sqrt_in_epoch
Where max_sqrt_in_epoch = max(sqrt(shares_raw_j) for all j)

Effect:

Miner	Raw Shares	√Shares	norm_shares
ASIC Farm	10,000	100	1.00
GPU Rig	2,500	50	0.50
CPU Miner	1,600	40	0.40
Despite 6.25× more raw shares, the ASIC farm has only 2.5× higher normalized contribution.

Reward Calculation
text
reward_i = (PoCI_i / Σ PoCI_j) × (EPOCH_REWARD + tx_fees)
Where EPOCH_REWARD is a constant parameter (to be defined for Zcash adaptation).

Complete Example
Miner	Shares	Loyalty	Bond	norm_shares	norm_loyalty	norm_bond	PoCI	Reward
ASIC Farm	10,000	100	5.0	1.00	1.00	1.00	1.0000	41.2%
GPU Rig	2,500	50	1.0	0.50	0.50	0.20	0.4400	18.1%
CPU Miner	1,600	10	0	0.40	0.10	0.00	0.2600	10.7%
Key observation: All miners receive rewards every epoch. No miner waits months for a block.

Share Synchronization Between Nodes
Share Pool
Each node maintains an in-memory share pool for the current epoch:

text
share_pool = {
    miner_id_1: [share1, share2, ...],
    miner_id_2: [share3, share4, ...],
}
Memory Usage: ~180 MB per epoch (1,000 miners × 1,000 shares).

Epoch Commit
At epoch end (block N+1439):

text
epoch_commit_root = Merkle(all_shares)
broadcast: epoch_commit { epoch_index, root, timestamp }
Conflict Resolution
If local root != received root:

Request missing shares via getshares

Majority vote from ≥5 peers determines correct root

Peers with persistent mismatches (>3 times in 10 epochs) are banned for 1 hour

Difficulty Adjustment
Parameters
Parameter	Value
Adjustment interval	every 120 blocks
Target interval time	7,200 seconds (120 × 60 sec)
Adjustment function	linear adjustment
Bounds	±25% per adjustment
Formula
text
new_target = old_target × (actual_time_span / 7200)
Where actual_time_span = timestamp[block_N+119] - timestamp[block_N]

Clamp:

If new_target > old_target × 1.25 → new_target = old_target × 1.25

If new_target < old_target × 0.75 → new_target = old_target × 0.75

Time-Warp Attack Protection
Blocks must satisfy: timestamp[i] > median(timestamp[i-1..i-11])

Transactions
Transaction Structure
Field	Type	Description
version	uint32	Transaction version
inputs	Vec<TxIn>	UTXOs being spent
outputs	Vec<TxOut>	New UTXOs
locktime	uint32	Block/time lock
TxIn (Input)
Field	Type	Description
prev_txid	[32]byte	Hash of previous transaction
prev_index	uint32	Output index
scriptSig	VarBytes	Unlocking script (signature + pubkey)
sequence	uint32	For relative timelocks
TxOut (Output)
Field	Type	Description
value	uint64	Amount
scriptPubKey	VarBytes	Locking script
Supported Script Types
Script	Description
P2PKH	Pay-to-Public-Key-Hash (Bitcoin standard)
P2PK	Pay-to-Public-Key
Multisig	M-of-N signatures
OP_CHECKLOCKTIMEVERIFY	Absolute timelocks
OP_CHECKSEQUENCEVERIFY	Relative timelocks
OP_BOND	Bond output (locked for 20,160 blocks)
OP_SLASH_EQUIVOCATION	Slashing transaction
Validation Rules
Rule	Description
Balance	∑inputs >= ∑outputs + fee
Minimum fee	Fee >= dust threshold
No double-spend	Same input cannot appear twice
Signature validation	ECDSA secp256k1
Epoch Lifecycle
Epoch Structure
Epoch N consists of blocks numbered [N×1440, N×1440+1439]

Epoch 1: blocks 0-1439

Epoch 2: blocks 1440-2879

Epoch 3: blocks 2880-4319

During Epoch
Miners:

Perform Argon2id(header)

If hash < target_block → block found, broadcast

If hash < target_share → share found, broadcast

Nodes:

Validate all blocks and shares

Maintain share pool for current epoch

Track loyalty of each miner

Track bond of each miner

At Epoch Boundary (Block 1440, 2880, ...)
Network stops accepting shares for epoch N

Calculate PoCI for each miner:

text
PoCI_i = 0.6×norm_shares_i + 0.2×norm_loyalty_i + 0.2×norm_bond_i
Calculate rewards:

text
reward_i = (PoCI_i / Σ PoCI_j) × (EPOCH_REWARD + tx_fees)
Generate payout transactions

Clear share pool

Update loyalty (decay for miners who missed epoch)

Adjust difficulty based on previous 120 blocks

Security Analysis
ASIC Resistance
Mechanism: Argon2id with 256 MB memory

Analysis:

ASIC would require 256 MB on-chip memory

Estimated development cost: $50,000+ per chip

CPU cost: $500 × 2 years = $1,000

Conclusion: ASICs are economically unviable. CPU remains competitive.

Peaceful Coexistence of ASIC and Small Miners
Mechanism: Epoch-based distribution + square-root normalization

Traditional PoW problem:

ASIC miner: 420 kH/s → finds blocks regularly

CPU miner: 0.1 kH/s → never finds a block

F-PoC solution:

Every miner receives rewards every epoch

Square-root normalization limits ASIC advantage

Small miners receive predictable daily income

Result: No miner type is driven out of the market.

Sybil Resistance
Mechanism: Minimum bond (1 unit)

Analysis:

Creating 1,000 miner_ids requires locking 1,000 units

Bond locked for 14 days = 14,000 unit-days of capital

Conclusion: Sybil attacks become expensive, cost scales with network value.

Anti-Burst Mining
Mechanism: Loyalty decay

Analysis:

Miner who mines only occasionally loses loyalty

Loyalty decreases by 30-50% for each missed epoch

After ~7 days absence, loyalty approaches zero

Conclusion: "Hit-and-run" strategies are ineffective. Long-term participation is rewarded.

51% Attack Protection
Requirements for success:

51% of all epoch shares (computational control)

High loyalty (long-term participation)

Significant bond (economic commitment at risk)

Conclusion: 51% attack in F-PoC is economically infeasible even for well-funded adversaries.

Implementation Notes
Constants
rust
// Time
TARGET_BLOCK_TIME = 60
EPOCH_BLOCKS = 1,440
EPOCH_DURATION = 86,400  // 24 hours

// Proof-of-Work
ARGON2_MEMORY = 268,435,456      // 256 MiB
ARGON2_ITERATIONS = 2
ARGON2_PARALLELISM = 4
ARGON2_VERSION = 0x13

// PoCI Weights
POCI_WEIGHT_SHARES = 0.6
POCI_WEIGHT_LOYALTY = 0.2
POCI_WEIGHT_BOND = 0.2

// Share Limits
MAX_SHARES_PER_MINER_PER_EPOCH = 5000

// Bond
MINIMUM_BOND = 1_000_000_000  // 1 unit in smallest denomination
BOND_LOCKUP_BLOCKS = 20,160   // ~14 days

// Difficulty
DIFFICULTY_ADJUSTMENT_INTERVAL = 120
TARGET_ADJUSTMENT_TIME = 7,200
MAX_DIFFICULTY_CHANGE = 0.25  // ±25%
Error Handling (Rust)
rust
fn add_balance(balance: &mut u64, amount: u64) -> Result<()> {
    *balance = balance.checked_add(amount)
        .ok_or(Error::Overflow)?;
    Ok(())
}
Data Types
All monetary amounts stored as uint64_t in smallest denomination

All integers serialized in little-endian

Variable-length data uses Bitcoin-style VarInt prefix

Hashes serialized as raw bytes

Appendices
Appendix A: Complete PoCI Calculation Example
Scenario: Epoch with 3 miners

Miner	Shares	Loyalty	Bond
A	10,000	100	5.0
B	2,500	50	1.0
C	1,600	10	0
Step 1: Normalize shares (square root)

max_sqrt = √10,000 = 100

norm_shares_A = 100/100 = 1.0

norm_shares_B = 50/100 = 0.5

norm_shares_C = 40/100 = 0.4

Step 2: Normalize loyalty

max_loyalty = 100

norm_loyalty_A = 100/100 = 1.0

norm_loyalty_B = 50/100 = 0.5

norm_loyalty_C = 10/100 = 0.1

Step 3: Normalize bond

max_bond = 5.0

norm_bond_A = 5.0/5.0 = 1.0

norm_bond_B = 1.0/5.0 = 0.2

norm_bond_C = 0

Step 4: Calculate PoCI

PoCI_A = 0.6×1.0 + 0.2×1.0 + 0.2×1.0 = 1.0

PoCI_B = 0.6×0.5 + 0.2×0.5 + 0.2×0.2 = 0.44

PoCI_C = 0.6×0.4 + 0.2×0.1 + 0 = 0.26

ΣPoCI = 1.70

Step 5: Calculate rewards (assuming EPOCH_REWARD = 72 units)

reward_A = (1.0/1.70) × 72 = 42.35

reward_B = (0.44/1.70) × 72 = 18.59

reward_C = (0.26/1.70) × 72 = 11.06

Appendix B: Argon2id Benchmarks
Hardware	Memory	Time	Throughput	Power
Intel i5-10400	256MB	112ms	8.9 H/s	8W
Intel i7-12700K	256MB	95ms	10.5 H/s	12W
AMD Ryzen 5 5600X	256MB	108ms	9.3 H/s	9W
AMD Ryzen 7 5700X	256MB	110ms	9.1 H/s	11W
NVIDIA RTX 3090	256MB	42ms	24 H/s	18W
Appendix C: Variance Analysis
Traditional PoW (Bitcoin/Zcash style):

Coefficient of variation (CV) > 1.0 for small miners

Expected waiting time for 0.01% miner: ~3.5 days with high variance

F-PoC:

Coefficient of variation ≈ 0.1

All miners receive rewards every 24 hours

Predictable income stream for all participants

Document Version: 1.0 (Research Prototype)
Last Updated: March 2026
Research Lead: Andrii Dumitro
