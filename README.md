# Graph Analysis for Anti-Money Laundering
## Data
The data can be found online using the following link:
  - [IBM Transactions for Anti Money Laundering (AML)](https://www.kaggle.com/datasets/ealtman2019/ibm-transactions-for-anti-money-laundering-aml)

The data is generated using a multi-agent virtual world model. All of the agents in the virtual world have actions governed by statistical distributions. Thus the model and data are NOT based on obfuscating or anonymizing real individuals. Everything is synthetic. More specifically the underlying model uses a virtual world of banks, individuals, and companies -- with individuals and companies buying items, and doing bank transfers to make payments, get supplies, pay salaries, etc. The underlying model has good and bad actors, with bad actors doing things like smuggling, extortion, illegal gambling, etc. The bad actors sometimes attempt to launder ill-gotten funds resulting in money-laundering transactions.

**Variant used**: HI-Small — HI = higher illicit ratio, Small = smallest of three sizes
- `HI-Small_Trans.csv` contains individual transactions, including the Timestamp,From Bank,Account,To Bank,Account,Amount Received,Receiving Currency,
Amount Paid,Payment Currency,Payment Format,Is Laundering
- `HI-Small_Patterns.txt` groups a subset of labelled transactions into
  simulated laundering patterns
---
## Eight Laundering Pattern Types in the Dataset
- **Fan-Out**: One account distributes funds to many recipients simultaneously. Characteristic of placement — converting cash into the financial system through many small transfers.
- **Fan-In**: Many accounts funnel funds into one. Characteristic of integration — consolidating laundered money before final extraction.
- **Cycle**: Money moves through a chain of accounts and returns to the origin. Characteristic of layering — obscuring the money trail through circular transfers.
- **Stack**: Sequential chain transfers, each slightly reducing the amount. Designed to break the audit trail across multiple hops.
- **Bipartite**: Two groups of accounts transact exclusively between each other in an alternating pattern.
- **Gather-Scatter**: Funds collected from many sources, held briefly, then redistributed — a combination of fan-in and fan-out.
- **Scatter-Gather**: Funds distributed to intermediaries, which then all send to a single destination.
- **Random**: Multi-hop chains with random intermediate accounts, designed to evade pattern-matching rules.
---

## Problem Statements
**Ground rules**
1. The graph is built without the Is Laundering label. The label is only used at the end, to score the rules.
2. No machine learning. Detection comes from graph structure (who pays whom, in which order, how much).
3. A rule only counts if it beats two non-graph baselines: random (0.10 % laundering) and “ACH format only” (0.72 %, ≈ 7×, covers 85 % of laundering).

**PS1: Fan-in / fan-out “smurfing” hubs**
- **Question**: Is laundering concentrated in accounts that collect money from, or spread it   to, unusually many counterparties in a short time?
- **Why**: This is the classic collection/distribution typology (FAN-IN, FAN-OUT, GATHER-SCATTER): many small senders feed one collector, or one account sprays money out.
- **Graph method**: Count distinct counterparties per account in 24–48 h windows, add a burstiness measure, then rank accounts against their peers.
- **Early signal**: Top 0.1–1 % of ACH accounts by degree: 7–25 % are laundering accounts, vs 2.3 % for ACH accounts overall (3–11×).
  
**PS2: Round-tripping: time-ordered money cycles**
- **Question**: When money leaves an account and comes back to it through a loop of other accounts, in time order, is that laundering?
- **Why**: The CYCLE typology: funds go round a loop and return to their origin looking “clean”. It is the core of the layering stage.
- **Graph method**: Find strongly connected components, enumerate cycles of length 3–6 (Johnson's algorithm), and keep only time-respecting cycles (each hop happens after the previous one).
- **Early signal**: Transactions on time-respecting ACH cycles are 22 % laundering (≈ 218× random, 30× the ACH baseline). Recall is only about 5 %, so the rule is precise but narrow.

**PS3: Layering chains and pass-through (mule) accounts**
- **Question**: Can we trace money that passes quickly through a chain of intermediary accounts while keeping most of its amount?
- **Why**: Chain/STACK typologies and money mules: an account receives money and forwards almost the same amount within hours, so it is a pipe, not a wallet.
- **Graph method**: Match each outflow to a recent inflow (within 72 h, keeping 80–100 % of the amount), then link the matched hops into time-respecting paths.
- **Early signal**: Pass-through ACH transfers are 4.0 % laundering (≈ 40× random, 5.5× the ACH baseline), with recall about 7 %. Whether longer chains are more suspicious is not yet tested.

**PS4: Split-and-merge: scatter-gather motifs**
- **Question**: Are two accounts linked by several parallel short paths through low-activity “middle” accounts a laundering signature?
- **Why**: The SCATTER-GATHER and BIPARTITE typologies: split a sum, route it through throw-away accounts, then merge it again.
- **Graph method**: Count 2-hop paths with a sparse matrix product A·K·A, where K keeps only low-degree intermediaries, then add a time-order check.
- **Early signa**: Pairs with ≥ 3 low-degree intermediaries: 44 accounts, and all 44 are laundering. With ≥ 2 intermediaries: 239 accounts, 23 % laundering. Very precise, but it covers few cases.

**PS5: One bank vs. the whole network**
- **Question**: How much laundering structure does a single bank miss when it sees only its own customers' transactions, compared with a shared (consortium) view?
- **Why**: Real banks see only their own side of the graph, and this dataset is rare in showing the whole ecosystem. The answer informs policy on inter-bank information sharing.
- **Graph method**: Re-run the PS1–4 rules on each bank's own view of the graph and compare with the full graph. Then plot recall against the number of banks that share data.
- **Early signal**: Only 3 of 39 laundering loop structures (7.7 %) are fully visible to any single bank, and a median of 3 banks is involved. One-hop rules (PS1) lose nothing, because every transaction of an account passes through its own bank. Multi-hop rules (PS2–4) lose most of their structure.
---

  

  
