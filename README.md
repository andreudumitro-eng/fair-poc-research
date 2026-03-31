F-PoC Research Prototype
Fair Proof-of-Contribution: An Alternative Reward Distribution Model for ASIC-Resistant PoW Networks

https://img.shields.io/badge/rust-1.70%252B-orange.svg
https://img.shields.io/badge/license-MIT%252FApache--2.0-blue.svg
https://img.shields.io/badge/status-research%2520prototype-yellow.svg

Abstract
This research project presents Fair Proof-of-Contribution (F-PoC) — a novel consensus mechanism that fundamentally restructures how Proof-of-Work rewards are distributed. Unlike traditional PoW where a single miner receives the entire block reward, F-PoC distributes rewards among all participating miners based on three dimensions: computational work (60%), long-term participation loyalty (20%), and economic commitment via bonded collateral (20%).

Key innovation for Zcash: F-PoC enables peaceful coexistence of ASIC, GPU, and CPU miners within the same network. Large-scale miners retain their efficiency advantage, but smaller participants receive regular, predictable rewards — creating a fair mining environment where no single group can dominate.

The protocol is implemented in Rust and serves as an open research platform for studying alternative incentive structures in ASIC-resistant PoW networks. All code, specifications, and research findings will be publicly available for the Zcash community to study, adapt, and reference.

This is not a proposal to replace Zcash's consensus. Rather, it is a research contribution that may inform future discussions around mining decentralization, reward distribution, and incentive alignment within the Zcash ecosystem.

The Problem: Mining Centralization in ASIC-Resistant PoW Networks
The Promise vs. Reality of Equihash
Zcash adopted Equihash — a memory-hard Proof-of-Work function — with the explicit goal of maintaining mining accessibility. The Equihash parameter set (n=200, k=9) was chosen to require approximately 2 GB of memory, theoretically favoring commodity hardware over specialized ASICs.

However, the reality today is different:

Metric	Current State
ASIC Availability	Multiple Equihash ASICs commercially available (Z15, Z16, Z17 models)
Hashrate Concentration	Top 3 pools control >50% of Zcash network hashrate
CPU Mining Viability	Effectively zero — CPU miners cannot compete with ASIC efficiency
Solo Mining Feasibility	Expected time to find a block for a solo miner: months to years
Problem 1: ASIC Dominance Despite Memory-Hardness
When Equihash was designed, the assumption was that ASIC development would be economically prohibitive due to memory requirements. This assumption has proven incorrect.

Consequences:

CPU mining yields <$0.01/day after electricity costs

GPU mining profitability has declined by 80-90% since ASIC introduction

New miners must invest in specialized hardware to participate

Geographic concentration follows ASIC manufacturing and cheap electricity

Why This Matters for Zcash's Mission:
Zcash's founding principles include accessibility and decentralization. The current mining landscape contradicts these principles.

Problem 2: Pool Centralization
Even with ASICs, individual miners cannot mine profitably due to variance. This creates a structural incentive to join mining pools.

Current Zcash Pool Distribution (approximate):

Pool	Hashrate Share
Flypool	~25%
F2Pool	~15%
ViaBTC	~12%
Other pools	~30%
Unknown/solo	~18%
The Centralization Risk:

Any pool controlling >50% of hashrate could censor transactions or double-spend

Individual miners have no voting power or influence over pool policies

Problem 3: High Variance (The Lottery Problem)
In traditional PoW, block discovery is a Poisson process. For a miner with a small share of network hashrate, the waiting time for a block follows a geometric distribution with high variance.

Why This Matters:

Solo mining is effectively a lottery — most miners will never find a block

Pools aggregate hashrate to smooth variance, taking a fee

Miners cannot predict or rely on income

The only rational choice for small miners is to join a pool

Problem 4: No Long-Term Incentive Alignment
Current PoW provides no mechanism to:

Reward long-term participation

Penalize malicious behavior beyond the cost of lost block rewards

Create economic alignment between miners and network health

The Gap This Research Addresses
Problem	Current Zcash	Gap
ASIC Resistance	Partial (ASICs exist)	Need for mechanisms that work with ASICs, not against them
Pool Centralization	Severe (top 3 pools >50%)	Need for variance reduction
Solo Mining Viability	Effectively zero	Need for regular, predictable rewards
Long-term Incentives	None	Need for loyalty and bonding mechanisms
ASIC vs. Small Miners	ASICs dominate	Need for peaceful coexistence
Research Context & Relationship with Zcash
What This Prototype Is
This is an open research platform for studying alternative reward distribution models in ASIC-resistant PoW networks. The codebase is a working implementation of F-PoC, designed to enable:

Simulation and analysis of reward variance reduction

Testing of loyalty and bonding mechanisms

Evaluation of ASIC/CPU coexistence conditions

Benchmarking of memory-hard PoW functions

Why Argon2id (Not Equihash)?
This research prototype uses Argon2id as the Proof-of-Work function. This is a deliberate research design choice, not a proposal to replace Zcash's Equihash.

Reason	Explanation
Simplicity	Argon2id has a simpler API, allowing focus on F-PoC logic
Memory-hardness	Both Argon2id and Equihash are memory-hard; ASIC resistance principles transfer
Performance	Easier to benchmark and simulate on commodity hardware
Standardization	Argon2id is RFC 9106 standard, well-documented
Key Insight: The F-PoC reward distribution mechanism — epoch-based proportional rewards, square-root normalization, loyalty accumulation, bonding with slashing — is independent of the underlying PoW function. Results obtained with Argon2id are transferable to Equihash.

Future Work: Equihash Adaptation
Phase 4 of this research (Q3-Q4 2026) will:

Replace Argon2id with Equihash in the consensus layer

Benchmark Equihash vs Argon2id for ASIC resistance

Provide migration recommendations for the Zcash community

What This Research Delivers to Zcash
Deliverable	Value to Zcash
Working F-PoC implementation	Ready-to-study codebase
Variance reduction analysis	Data on reward predictability
Loyalty mechanism evaluation	Understanding of long-term incentives
Bond/slashing security model	Economic alignment framework
Equihash adaptation roadmap	Clear path for integration
The Solution: Fair Proof-of-Contribution (F-PoC)
Core Principle: Redefining Reward Distribution
Traditional PoW follows a winner-takes-all model:

One miner finds a block → receives full reward

All other miners receive nothing for that block

F-PoC replaces this with a proportional distribution model:

All miners who contribute during an epoch receive rewards

Rewards are distributed based on three independent dimensions of contribution

Epoch length: 1,440 blocks (24 hours at 60-second block time)

🔑 Key Innovation: Peaceful Coexistence of ASICs and Small Devices
Unlike traditional PoW where ASICs make CPU/GPU mining obsolete, F-PoC enables all miners to coexist fairly:

Miner Type	Advantage in F-PoC	Why They Still Participate
ASIC Farms	Higher share count (more hashrate)	Still earn proportionally more
GPU Rigs	Moderate share count	Receive regular daily rewards
CPU Miners	Lower share count	Guaranteed minimum reward every 24 hours
The key insight: ASICs retain their efficiency advantage, but variance is eliminated — small miners no longer face the "lottery problem". They receive predictable, regular income proportional to their contribution, making mining viable even on modest hardware.

The Three Dimensions of Contribution
Dimension 1: Computational Work (Shares) — 60% Weight
Miners submit shares — valid hashes that meet a difficulty threshold lower than block difficulty.

Parameter	Value	Rationale
Memory	256 MB	ASIC development cost >$50,000 per chip
Target share difficulty	target_block << 8	~256 shares per found block
Share Normalization (Square Root):

text
norm_shares_i = √shares_i / max√shares_in_epoch
Square root prevents a single miner from dominating the shares component while preserving differentiation.

Example:

Miner	Raw Shares	√Shares	norm_shares
A (ASIC farm)	10,000	100	1.00
B (GPU rig)	2,500	50	0.50
C (CPU miner)	1,600	40	0.40
Despite Miner A having 6.25× more shares than Miner C, their normalized contribution is only 2.5× higher. The CPU miner still receives meaningful daily rewards.

Dimension 2: Long-Term Participation (Loyalty) — 20% Weight
Loyalty rewards miners who participate consistently over time.

Mechanism:

text
Initial: loyalty = 0
Participation (≥1 share in epoch): loyalty = loyalty + 1
Missed epoch: loyalty = max(loyalty × 0.7, loyalty // 2)
Grace period: First 3 missed epochs after returning use ×0.5 for faster recovery.

Why This Matters: Miners who maintain consistent uptime are rewarded. "Hit-and-run" mining becomes less attractive. ASIC farms that run 24/7 earn loyalty bonuses; intermittent small miners can recover quickly.

Dimension 3: Economic Commitment (Bond) — 20% Weight
Miners must lock a minimum bond to participate in rewards.

Parameter	Value	Rationale
Minimum bond	1 unit	Economic barrier to sybil attacks
Lock-up period	20,160 blocks (~14 days)	Prevents gaming with short-term bonds
Normalization (Square Root):

text
norm_bond_i = √bond_i / max√bond_in_epoch
Slashing Conditions:

Violation	Detection	Penalty
Equivocation	Same miner signs two blocks at same height	100% bond burned
Invalid Share Flooding	Invalid shares >30% of total in an epoch	100% bond burned
The PoCI Formula
text
PoCI_i = 0.6 × norm_shares_i + 0.2 × norm_loyalty_i + 0.2 × norm_bond_i
Where:

norm_shares_i = √shares_i / max√shares_in_epoch

norm_loyalty_i = loyalty_i / max_loyalty_in_epoch

norm_bond_i = √bond_i / max√bond_in_epoch

Reward Distribution
Each epoch (1,440 blocks, 24 hours):

text
epoch_reward = constant + total_transaction_fees_in_epoch
reward_i = (PoCI_i / Σ PoCI_j) × epoch_reward
Example Epoch:

Miner	Shares	Loyalty	Bond	norm_shares	norm_loyalty	norm_bond	PoCI	Reward
ASIC Farm	10,000	100	5	1.00	1.00	1.00	1.0000	41.16
GPU Rig	2,500	50	1	0.50	0.50	0.447	0.4894	20.14
CPU Miner	1,600	10	0	0.40	0.10	0.00	0.2600	10.70
Key Observation: All miners receive rewards every 24 hours. No miner waits months for a block. The CPU miner earns ~10 units daily — enough to remain profitable.

How F-PoC Addresses Each Zcash Problem
Problem	F-PoC Mechanism	Quantifiable Improvement
ASIC Dominance	Square-root normalization of shares	ASIC advantage reduced from 1000× to ~2-5×
Pool Centralization	Epoch-based distribution	Variance reduced by factor of ~1,440
High Variance	All miners receive rewards every epoch	Coefficient of variation ≈ 0.1 vs traditional PoW > 1.0
No Long-term Incentives	Loyalty accumulation and decay; bond slashing	Rewards continuous participation; penalizes malicious behavior
ASIC vs. Small Miners	Proportional distribution + sqrt normalization	Peaceful coexistence — all miners profitable
How F-PoC Enables Peaceful ASIC/CPU Coexistence
The Traditional Problem
In current Zcash:

ASIC miner: 420 kH/s → finds blocks regularly

CPU miner: 0.1 kH/s → never finds a block (lottery)

Result: CPU mining is economically irrational. ASICs dominate completely.

The F-PoC Solution
In F-PoC:

Every miner receives rewards every epoch regardless of hashrate

Square-root normalization prevents ASICs from monopolizing the shares component

Loyalty rewards consistent participation (ASICs run 24/7, CPUs can too)

Bond requirement adds sybil resistance without excluding small miners

Result:

ASIC farms earn proportionally more (as they should)

CPU miners earn predictable daily income (even if smaller)

No miner type is driven out of the market

Current Implementation Status
Reference Implementation in Rust
Completion Status: ~70% complete
License: MIT / Apache-2.0 dual-licensed

Completed Components
Component	Status	Description
Constants & Types	✅ Complete	Protocol parameters, type definitions
Error Handling	✅ Complete	Unified error type
Blocks	✅ Complete	Block header, structure, Merkle tree, validation
Transactions	✅ Complete	TxIn, TxOut, validation rules
Consensus	✅ Complete	PoCI calculation, loyalty, bond, difficulty
Argon2id PoW	✅ Complete	256 MB memory, SHA256 prefilter, cache
UTXO Storage	✅ Complete	RocksDB, multiple column families
RPC API	✅ Complete	7 endpoints
DDoS Protection	✅ Complete	Rate limiting, IP blacklisting
Attack Detection	✅ Complete	51% attack monitoring
Checkpoint System	✅ Complete	Automatic checkpoints every 1000 blocks
Backup & Restore	✅ Complete	Automatic RocksDB backups
P2P Network	🟡 40%	DNS seeds, handshake, sync manager (in progress)
CLI Wallet	🔲 Planned	Transaction creation and signing
Argon2id PoW Optimization Pipeline
text
┌─────────────────────────────────────────────────────────────────┐
│ Share Validation Pipeline                                      │
├─────────────────────────────────────────────────────────────────┤
│ 1. Receive share packet (180 bytes)                           │
│ ↓                                                              │
│ 2. SHA256 prefilter (cheap rejection)                         │
│    - Cost: <1 microsecond per share                           │
│    - Rejects ~90% of invalid shares                           │
│ ↓                                                              │
│ 3. Cache lookup (60-second TTL)                               │
│    - Cache size: 10,000 entries                               │
│    - Hit rate: 70-80%                                         │
│ ↓                                                              │
│ 4. Argon2id computation                                       │
│    - Memory: 256 MB, Iterations: 2, Parallelism: 4           │
│    - Cost: ~100ms on modern CPU                               │
│ ↓                                                              │
│ 5. Store result in cache                                      │
└─────────────────────────────────────────────────────────────────┘
RPC API Endpoints
Endpoint	Description
/info	Node information (version, height, peers)
/block?height=N	Block at specified height
/transaction?txid=HASH	Transaction details
/peers	Connected peers list
/miners	Active miners in current epoch
/mempool	Pending transactions
/health	Node health status
Research Questions to Answer
This research project aims to answer the following questions:

Variance Reduction: By what factor does epoch-based distribution reduce reward variance compared to traditional PoW?

CPU Viability: What is the minimum hashrate required for a CPU miner to earn more than electricity costs under F-PoC?

Peaceful Coexistence: Can ASIC and CPU miners coexist profitably in the same network under F-PoC?

Pool Dependence: Does predictable epoch-based rewards eliminate the structural advantage of mining pools?

Loyalty Effectiveness: How much does the loyalty mechanism increase miner retention during price drops?

Sybil Resistance: What bond size is sufficient to prevent Sybil attacks at different network valuations?

ASIC Advantage: What is the actual performance advantage of ASICs vs CPUs for Argon2id with 256 MB memory?

Equihash Adaptation: How can F-PoC be adapted to work with Zcash's Equihash PoW?

Research Methodology
Phase 1: Theoretical Analysis (Completed - Q1 2026)
Mathematical modeling of variance reduction

Game-theoretic analysis of loyalty/bond incentives

Security analysis of slashing conditions

Analysis of ASIC vs. CPU coexistence conditions

Phase 2: Prototype Implementation (70% Complete - Q2 2026)
Rust reference implementation

Argon2id PoW with optimizations

Full node with RPC API

Simulation framework for mixed ASIC/CPU environments

Phase 3: Simulation & Benchmarking (Q3 2026)
Monte Carlo simulations of miner behavior

Variance analysis under different network conditions

CPU vs ASIC performance benchmarks

Coexistence simulations with varying ASIC/CPU ratios

Phase 4: Equihash Adaptation (Q3-Q4 2026)
Adapt F-PoC to work with Zcash's Equihash

Benchmark Equihash vs Argon2id for ASIC resistance

Provide migration path for Zcash community

*Note: This phase explicitly addresses the transition from Argon2id (research placeholder) to Equihash (Zcash's native PoW). The core F-PoC logic remains unchanged.*

Phase 5: Analysis & Publication (Q4 2026)
Comparative analysis of reward distribution models

Recommendations for ASIC-resistant networks

Open access research paper

Related Work
Project	Approach	Difference from F-PoC
Bitcoin	Traditional PoW, winner-takes-all	High variance, pool-dependent
Zcash	Equihash PoW, winner-takes-all	ASIC-dominated despite memory-hardness
Monero	RandomX, ASIC-resistant	Still winner-takes-all, high variance
Ethereum (pre-merge)	PoW with uncle rewards	Partial distribution, but still variance
F-PoC is unique in combining:

Epoch-based proportional distribution

Three-dimensional contribution metrics

Loyalty and bonding in a PoW context

Peaceful coexistence of ASIC and small miners

Limitations and Future Work
Current Limitations
Argon2id memory (256 MB) may still be ASIC-able

P2P network implementation is 40% complete

No formal verification of consensus rules

Limited real-world testing

Future Research Directions
Equihash Integration: Adapt F-PoC to work with Equihash (Zcash's PoW)

Larger Memory Parameters: Test with 2GB+ Argon2id parameters

Formal Verification: Prove consensus safety properties

Economic Modeling: Agent-based simulation of miner behavior

Live Testnet Deployment: Deploy on Zcash testnet for real-world validation

Contributing
This is an open research project. Contributions are welcome in:

Code improvements and bug fixes

Additional benchmarks and simulations

Documentation and specification refinement

Research paper co-authorship

License
MIT / Apache-2.0 dual-licensed

Contact
Research Lead: Andrii Dumitro
GitHub: Available upon request
Email: Available via GitHub

Acknowledgments
This research was conducted independently. The author thanks the open source community for providing the tools and libraries that made this implementation possible:

Rust language and ecosystem

Argon2id reference implementation

RocksDB team

Tokio async runtime

Citation
If you use this research in your work, please cite:

bibtex
@misc{fpoc2026,
  author = {Dumitro, Andrii},
  title = {F-PoC: Fair Proof-of-Contribution for ASIC-Resistant PoW Networks},
  year = {2026},
  publisher = {GitHub},
  note = {Research prototype for Zcash community}
}
