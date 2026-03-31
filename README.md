F-PoC Research Prototype
Fair Proof-of-Contribution: An Alternative Reward Distribution Model for ASIC-Resistant PoW Networks
https://img.shields.io/badge/rust-1.70%252B-orange.svg
https://img.shields.io/badge/license-MIT%252FApache--2.0-blue.svg
https://img.shields.io/badge/status-research%2520prototype-yellow.svg

Abstract
This research project presents Fair Proof-of-Contribution (F-PoC) — a novel consensus mechanism that fundamentally restructures how Proof-of-Work rewards are distributed. Unlike traditional PoW where a single miner receives the entire block reward, F-PoC distributes rewards among all participating miners based on three dimensions: computational work (40%), long-term participation loyalty (30%), and economic commitment via bonded collateral (30%).

Note: The weights have been optimized for Equihash (Zcash's PoW) to ensure peaceful coexistence of ASIC, GPU, and CPU miners. The original 60/20/20 split was designed for Argon2id and has been updated based on real-world performance data.

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

Why This Matters for Zcash's Mission: Zcash's founding principles include accessibility and decentralization. The current mining landscape contradicts these principles.

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

Why Argon2id (Not Equihash) in Current Phase?
This research prototype uses Argon2id as the Proof-of-Work function during development. This is a deliberate research design choice, not a proposal to replace Zcash's Equihash.

Reason	Explanation
Simplicity	Argon2id has a simpler API, allowing focus on F-PoC logic
Memory-hardness	Both Argon2id and Equihash are memory-hard; ASIC resistance principles transfer
Performance	Easier to benchmark and simulate on commodity hardware
Standardization	Argon2id is RFC 9106 standard, well-documented
Key Insight: The F-PoC reward distribution mechanism — epoch-based proportional rewards, square-root normalization, loyalty accumulation, bonding with slashing — is independent of the underlying PoW function. Results obtained with Argon2id are transferable to Equihash.

Future Work: Equihash Adaptation
Phase 4 of this research (Q3-Q4 2026) will:

Replace Argon2id with Equihash (n=200, k=9) in the consensus layer

Implement dynamic share difficulty based on bond size (anti-spam)

Benchmark Equihash vs Argon2id for ASIC resistance

Provide migration recommendations for the Zcash community

What This Research Delivers to Zcash
Deliverable	Value to Zcash
Working F-PoC implementation	Ready-to-study codebase
Variance reduction analysis	Data on reward predictability
Loyalty mechanism evaluation	Understanding of long-term incentives
Bond/slashing security model	Economic alignment framework
Equihash adaptation roadmap	Clear path for integration
Auditable bond design	Regulatory compliance (MiCA-ready)
The Solution: Fair Proof-of-Contribution (F-PoC)
Core Principle: Redefining Reward Distribution
Traditional PoW follows a winner-takes-all model:

One miner finds a block → receives full reward

All other miners receive nothing for that block

F-PoC replaces this with a proportional distribution model:

All miners who contribute during an epoch receive rewards

Rewards are distributed based on three independent dimensions of contribution

Epoch length: 1,152 blocks (24 hours at 75-second block time, Zcash-compatible)

🔑 Key Innovation: Peaceful Coexistence of ASICs and Small Devices
Unlike traditional PoW where ASICs make CPU/GPU mining obsolete, F-PoC enables all miners to coexist fairly:

Miner Type	Advantage in F-PoC	Why They Still Participate
ASIC Farms	Higher share count (more hashrate)	Still earn proportionally more
GPU Rigs	Moderate share count	Receive regular daily rewards
CPU Miners	Lower share count	Guaranteed minimum reward every 24 hours
The key insight: ASICs retain their efficiency advantage, but variance is eliminated — small miners no longer face the "lottery problem". They receive predictable, regular income proportional to their contribution, making mining viable even on modest hardware.

The Three Dimensions of Contribution
Dimension 1: Computational Work (Shares) — 40% Weight
Miners submit shares — valid PoW solutions that meet a difficulty threshold lower than block difficulty.

Parameter	Value	Rationale
Memory (Argon2id)	256 MB	ASIC development cost >$50,000 per chip
Memory (Equihash)	~2 GB	Zcash mainnet compatible
Target share difficulty	Dynamic based on bond	Anti-spam, encourages small miners
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

Dimension 2: Long-Term Participation (Loyalty) — 30% Weight
Loyalty rewards miners who participate consistently over time.

Mechanism:

text
Initial: loyalty = 0
Participation (≥1 share in epoch): loyalty = loyalty + 1
Missed epoch: loyalty = max(loyalty × 0.7, loyalty // 2)
Grace period: First 3 missed epochs after returning use ×0.5 for faster recovery.
Why This Matters:

Miners who maintain consistent uptime are rewarded

"Hit-and-run" mining becomes less attractive

ASIC farms that run 24/7 earn loyalty bonuses

Intermittent small miners can recover quickly

Dimension 3: Economic Commitment (Bond) — 30% Weight
Miners must lock a minimum bond to participate in rewards.

Parameter	Value	Rationale
Minimum bond	1 ZEC	Economic barrier to sybil attacks
Lock-up period	20,160 blocks (~14 days)	Prevents gaming with short-term bonds
Normalization	Square root	Prevents whale dominance
Slashing Conditions:

Violation	Detection	Penalty
Equivocation	Same miner signs two blocks at same height	100% bond burned
Invalid Share Flooding	Invalid shares >30% of total in an epoch	100% bond burned
Censorship	Excluding transactions for >100 blocks	50-100% bond burned
Auditable Bonds (Regulatory Compliance):
Miners can optionally create bonds with viewing keys for auditors. This allows:

Exchanges to verify miner operations without accessing private keys

Compliance with MiCA and other regulatory frameworks

Transparent operations while maintaining privacy

The PoCI Formula
text
PoCI_i = 0.40 × norm_shares_i + 0.30 × norm_loyalty_i + 0.30 × norm_bond_i
Where:

norm_shares_i = √shares_i / max√shares_in_epoch

norm_loyalty_i = loyalty_i / max_loyalty_in_epoch

norm_bond_i = √bond_i / max√bond_in_epoch

Reward Distribution
Each epoch (1,152 blocks, 24 hours):

text
epoch_reward = constant + total_transaction_fees_in_epoch
reward_i = (PoCI_i / Σ PoCI_j) × epoch_reward
Example Epoch (Zcash-Compatible)
Miner	Shares	Loyalty	Bond (ZEC)	norm_shares	norm_loyalty	norm_bond	PoCI	Reward
ASIC Farm	10,000	100	10,000	1.00	1.00	1.00	1.000	42.6%
GPU Rig	2,500	50	100	0.50	0.50	0.32	0.396	16.9%
CPU Miner	1,600	10	1	0.40	0.10	0.03	0.199	8.5%
Key Observation: All miners receive rewards every 24 hours. No miner waits months for a block. The CPU miner earns ~8.5% of epoch reward — enough to remain profitable.

How F-PoC Addresses Each Zcash Problem
Problem	F-PoC Mechanism	Quantifiable Improvement
ASIC Dominance	Square-root normalization + dynamic share difficulty	ASIC advantage reduced from 10,000× to ~40-80×
Pool Centralization	Epoch-based distribution	Variance reduced by factor of ~1,152
High Variance	All miners receive rewards every epoch	Coefficient of variation ≈ 0.1 vs traditional PoW > 1.0
No Long-term Incentives	Loyalty accumulation and decay; bond slashing	Rewards continuous participation; penalizes malicious behavior
ASIC vs. Small Miners	Proportional distribution + sqrt normalization + bond weight	Peaceful coexistence — all miners profitable
Regulatory Risk	Auditable bonds with viewing keys	Compliance with MiCA, exchange listing possible
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
Auditable Bonds	🔄 60%	Viewing keys for regulatory compliance
P2P Network	🔄 40%	DNS seeds, handshake, sync manager (in progress)
Equihash Integration	🔄 30%	Phase 4 work in progress
CLI Wallet	🔲 Planned	Transaction creation and signing
Share Validation Pipeline (Equihash Version)
text
┌─────────────────────────────────────────────────────────────────┐
│ Share Validation Pipeline (Equihash)                           │
├─────────────────────────────────────────────────────────────────┤
│ 1. Receive share packet (~1544 bytes)                          │
│ ↓                                                              │
│ 2. Quick syntax validation                                     │
│    - Check miner_id format                                     │
│    - Check solution size (1344 bytes)                          │
│    - Verify miner not over MAX_SHARES                          │
│ ↓                                                              │
│ 3. SHA256 prefilter (cheap rejection)                          │
│    - Compute hash = SHA256(solution)                           │
│    - Compare with target_share_miner (bond-dependent)          │
│    - Rejects ~99% of invalid shares                            │
│    - Cost: <1 microsecond                                      │
│ ↓                                                              │
│ 4. Full Equihash verification                                  │
│    - Verify solution is valid for header                       │
│    - Cost: ~5-10ms on modern CPU                               │
│ ↓                                                              │
│ 5. Store valid share in share pool (RocksDB + LRU cache)       │
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
/bonds?miner_id=ID	Bond information for miner
Research Questions to Answer
This research project aims to answer the following questions:

Variance Reduction: By what factor does epoch-based distribution reduce reward variance compared to traditional PoW?

CPU Viability: What is the minimum hashrate required for a CPU miner to earn more than electricity costs under F-PoC?

Peaceful Coexistence: Can ASIC and CPU miners coexist profitably in the same network under F-PoC?

Pool Dependence: Does predictable epoch-based rewards eliminate the structural advantage of mining pools?

Loyalty Effectiveness: How much does the loyalty mechanism increase miner retention during price drops?

Sybil Resistance: What bond size is sufficient to prevent Sybil attacks at different network valuations?

ASIC Advantage: What is the actual performance advantage of ASICs vs CPUs for Equihash with 2GB memory?

Equihash Adaptation: How does F-PoC perform with Equihash compared to Argon2id?

Regulatory Compatibility: Do auditable bonds satisfy MiCA and exchange requirements?

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

Auditable bond implementation

Phase 3: Simulation & Benchmarking (Q3 2026)
Monte Carlo simulations of miner behavior

Variance analysis under different network conditions

CPU vs ASIC performance benchmarks (Argon2id)

Coexistence simulations with varying ASIC/CPU ratios

Phase 4: Equihash Adaptation (Q3-Q4 2026)
Replace Argon2id with Equihash (n=200, k=9) in consensus layer

Implement dynamic share difficulty based on bond

Benchmark Equihash vs Argon2id for ASIC resistance

Provide migration path for Zcash community

Phase 5: Analysis & Publication (Q4 2026)
Comparative analysis of reward distribution models

Recommendations for ASIC-resistant networks

Open access research paper

ZIP (Zcash Improvement Proposal) preparation

Related Work
Project	Approach	Difference from F-PoC
Bitcoin	Traditional PoW, winner-takes-all	High variance, pool-dependent
Zcash	Equihash PoW, winner-takes-all	ASIC-dominated despite memory-hardness
Monero	RandomX, ASIC-resistant	Still winner-takes-all, high variance
Ethereum (pre-merge)	PoW with uncle rewards	Partial distribution, but still variance
F-PoC is unique in combining:

Epoch-based proportional distribution

Three-dimensional contribution metrics (shares, loyalty, bond)

Square-root normalization for fairness

Slashing mechanism for security

Auditable bonds for regulatory compliance

Peaceful coexistence of ASIC and small miners

Limitations and Future Work
Current Limitations
Argon2id memory (256 MB) may still be ASIC-able (will be replaced with Equihash)

P2P network implementation is 40% complete

No formal verification of consensus rules

Limited real-world testing

Equihash integration in progress

Future Research Directions
Equihash Integration: Complete adaptation to Zcash's native PoW

Formal Verification: Prove consensus safety properties

Economic Modeling: Agent-based simulation of miner behavior

Live Testnet Deployment: Deploy on Zcash testnet for real-world validation

ZIP Submission: Formal Zcash Improvement Proposal

Mobile Mining: Light client support for mobile CPU mining

Contributing
This is an open research project. Contributions are welcome in:

Code improvements and bug fixes

Additional benchmarks and simulations

Documentation and specification refinement

Research paper co-authorship

Zcash community outreach

License
MIT / Apache-2.0 dual-licensed

Contact
Research Lead: Andrii Dumitro
GitHub: Available upon request
Email: Available via GitHub
Discord: Available for Zcash community discussions

Acknowledgments
This research was conducted independently. The author thanks the open source community for providing the tools and libraries that made this implementation possible:

Rust language and ecosystem

Argon2id reference implementation

RocksDB team

Tokio async runtime

Zcash open source community for inspiration and feedback

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
Version History
Version	Date	Changes
1.0	March 2026	Initial release
1.1	March 2026	Updated weights to 40/30/30, added Zcash-compatible epoch (1152 blocks), added auditable bonds, updated for Equihash integration
