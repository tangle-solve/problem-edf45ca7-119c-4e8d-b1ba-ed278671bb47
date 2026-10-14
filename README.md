<!-- solve-problem:edf45ca7-119c-4e8d-b1ba-ed278671bb47 -->
# Fast conflict resolution for SPECTRE-style ledgers

A [Tangle Solve problem](https://solve.tangle.tools/problems/edf45ca7-119c-4e8d-b1ba-ed278671bb47).

## What is the problem?

SPECTRE (Sompolinsky, Lewenberg, Zohar, 2016) is a Proof-of-Work consensus protocol whose underlying data structure is a block DAG rather than a chain. Blocks may reference multiple predecessors; each block casts a "vote" in pairwise elections between any two conflicting transactions, and the protocol outputs a **pairwise majority relation** over transactions rather than a total order. This yields very fast confirmation for *independent* transactions but, because the majority relation can contain cycles, SPECTRE cannot in general serialize conflicting/dependent transactions — it only guarantees eventual resolution in a restricted "asynchronous" fault model where attackers must publish blocks referencing their parents honestly.

The open problem: **design a DAG-ledger protocol that (a) retains SPECTRE-like sub-minute (fast, high-throughput) confirmation for independent transactions; (b) guarantees bounded-time or probabilistically bounded resolution for conflicting/dependent transaction sets; and (c) provides an explicitly priced total-order "fallback" — a mechanism by which users can pay (fee, delay, or security margin) to obtain a total order over a set of transactions, restoring serializability for smart-contract-style workloads.**

Formal setting: blocks arrive as a PoW DAG G; adversary controls hash power β < 1/2 (SPECTRE tolerates up to 50% under its fault model). Desired outputs: (1) a confirmation rule with provable pre-confirmation latency bounds for pairwise-disjoint transactions; (2) a resolution rule for conflicting sets with a stated convergence time (in blocks/time, as a function of β); (3) a fallback ordering rule that is either explicitly priced via fees or bounded via a safety-parameter tradeoff. The precise formalization of "bounded" and "priced" is itself part of the problem — reasonable interpretations: expected block-count bound on conflict resolution, and a fee market or auction determining total-ordering priority.

## Why does it matter?

SPECTRE-family protocols (SPECTRE, PHANTOM, GHOSTDAG) achieve high throughput and fast confirmation but explicitly refuse smart-contract compatibility: any application needing serialization (payments with dependencies, DeFi, state machines) cannot use them. A solution would unlock fast, scalable general-purpose blockchains on PoW (and the analysis transfers to DAG-structured PoS committees), giving block-DAG systems the expressiveness of account-model chains without sacrificing latency. It would also formally resolve the standing gap between SPECTRE (fast, weak ordering) and PHANTOM/GHOSTDAG (total order, slower confirmation).

## What's already known?

- **SPECTRE (2016)**: gives the pairwise-voting DAG protocol; proves safety/liveness for independent transactions under an asynchronous adversary limited to "honest block referencing" (blocks must be published referencing all observed tips). Its authors explicitly acknowledge it does not provide a total order; confirming conflicting transactions requires waiting for the vote gap to stabilize, with no tight bound beyond asymptotic eventual resolution.
- **PHANTOM (Sompolinsky & Zohar) and GHOSTDAG (Bagaria et al., arXiv:1810.08092, 2019)**: solve the ordering problem by outputting a total order via clustering the DAG into a well-formed "blue set" (parameter k) and ordering it topologically. This restores serializability but slows confirmation toward Nakamoto-like latency, since a block must wait to be robustly classified into the blue set. That is precisely the tradeoff this problem asks to beat: keep SPECTRE speed for independent transactions *and* get ordering when needed.
- No published protocol combines (a)–(c) above; existing systems (e.g., Kaspr/GHOSTDAG deployments, Prism-style work by Bagaria et al. on transaction separation) attack throughput/latency separation but not a priced, bounded total-order fallback layered on pairwise voting. (Verify current literature — this is the frontier as recorded.)

## What would count as solving it?

Any of the following, with proofs:

1. **Full solution**: A protocol Π on the PoW DAG with (i) a theorem bounding expected confirmation latency for pairwise-independent transactions matching SPECTRE's, (ii) a theorem giving an explicit bound (function of β and a user-chosen parameter) on the time/blocks to resolve any conflicting set, and (iii) a formally specified fallback total-order mechanism with an explicit price (fee schedule or delay/security parameter) and a proof the order is safe.
2. **Impossibility/lower bound**: A proof that no DAG protocol can simultaneously achieve SPECTRE-latency for independent transactions and any nontrivial bounded conflict resolution under the same adversary model — this would close the problem negatively and justify hybrid architectures.
3. **Concrete instantiation**: A construction (e.g., two-tier DAG: SPECTRE votes for fast path, occasional "ordering blocks" provably finalizing a total order) with simulation or formal analysis demonstrating the latency/pricing frontier, plus security proof.

Referee criterion: formal latency and safety theorems, not just system description.

## How to start

1. Read SPECTRE (ePrint 2016/1159), Sections on the voting protocol and the discussion of its ordering limitation; then GHOSTDAG (arXiv:1810.08092) for the total-ordering machinery (blue sets, k-cluster) you'll borrow for the fallback.
2. Natural approach: **hybrid/two-tier** — run SPECTRE pairwise voting as the fast path; add a periodic or fee-triggered "ordering epoch" whose blocks commit to a topological order of currently-unresolved conflicts, using GHOSTDAG's blue-set ordering restricted to conflicted vertices. Prove the fast path's latency is unaffected (ordering blocks are a low-rate sidecar) and derive the resolution bound from vote-gap martingale arguments as in SPECTRE's robustness lemma.
3. Formalize "bounded": work in the asynchronous honest-referencing adversary model of SPECTRE; state the resolution bound as a high-probability block-count bound.
4. Main obstacle: the adversarial model — if you strengthen the adversary beyond SPECTRE's, safety may fail; if you keep it, reviewers will ask whether the fallback can be censorship-delayed. Address fee-based ordering incentives explicitly (game-theoretic as well as cryptographic analysis).

## Key references

- Y. Sompolinsky, Y. Lewenberg, A. Zohar, "SPECTRE: Serialization of Proof-of-work Events: Confirming Transactions via Recursive Elections," IACR ePrint 2016/1159. https://eprint.iacr.org/2016/1159 — Defines the block-DAG pairwise-voting protocol with fast confirmation for independent transactions and explicitly notes the lack of total ordering.
- V. Bagaria, S. Deb, S. Kannan, D. Tse, "Prism: Deconstructing the Blockchain to Approach Physical Limits of Transaction Throughput and Propagation Delay" (and the GHOSTDAG analysis lineage), arXiv:1810.08092. https://arxiv.org/abs/1810.08092 — Separates transaction/committee/ proposer block types on a DAG to push throughput and latency toward physical limits; relevant architectural template for tiered DAG designs. (Verify which of Prism/GHOSTDAG this preprint version corresponds to.)
- Sompolinsky & Zohar's PHANTOM/GHOSTDAG papers (IACR ePrint, ~2018; verify exact number) — Provide the k-cluster blue-set construction yielding a provably-bounded total order on a block DAG, at the cost of slower confirmation.

## Warnings

- SPECTRE's safety proofs rely on an unusual adversary restriction (must reference known blocks when publishing); naively adding ordering machinery may invalidate proofs — recheck the model.
- "Bounded conflict resolution" is not formally defined in the census record; you must choose and justify a definition (high-probability block bound vs. expected time vs. fee-purchased deadline).
- A common dead end: trying to extract a total order from the SPECTRE majority relation directly — it can cycle, and topological shortcuts reintroduce GHOSTDAG-style latency.
- The Prism arXiv number cited in the census may conflate Prism and GHOSTDAG; verify before citing.

---
*`BCGT-000257` · flagged for review*

## Acceptance guidance

Any of the following, with proofs:

1. **Full solution**: A protocol Π on the PoW DAG with (i) a theorem bounding expected confirmation latency for pairwise-independent transactions matching SPECTRE's, (ii) a theorem giving an explicit bound (function of β and a user-chosen parameter) on the time/blocks to resolve any conflicting set, and (iii) a formally specified fallback total-order mechanism with an explicit price (fee schedule or delay/security parameter) and a proof the order is safe.
2. **Impossibility/lower bound**: A proof that no DAG protocol can simultaneously achieve SPECTRE-latency for independent transactions and any nontrivial bounded conflict resolution under the same adversary model — this would close the problem negatively and justify hybrid architectures.
3. **Concrete instantiation**: A construction (e.g., two-tier DAG: SPECTRE votes for fast path, occasional "ordering blocks" provably finalizing a total order) with simulation or formal analysis demonstrating the latency/pricing frontier, plus security proof.

Referee criterion: formal latency and safety theorems, not just system description.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/edf45ca7-119c-4e8d-b1ba-ed278671bb47?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000257.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000257)
