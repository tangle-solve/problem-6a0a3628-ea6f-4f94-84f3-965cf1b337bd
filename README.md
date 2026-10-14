<!-- solve-problem:6a0a3628-ea6f-4f94-84f3-965cf1b337bd -->
# Proof-producing attack discovery

A [Tangle Solve problem](https://solve.tangle.tools/problems/6a0a3628-ea6f-4f94-84f3-965cf1b337bd).

## What is the problem?

Selfish-mining analysis is typically done by modeling the mining game as a Markov Decision Process (MDP) whose states encode the attacker's private chain length, the public chain length, and (in richer models) continuous quantities such as time since last block, network difficulty adjustment variables, or DAG-local views. The optimal policy of this MDP yields the maximum relative revenue of a deviating miner, i.e., an attack profitability bound.

The open problem: given an *approximate, learned or numerically computed* mining policy and value function (e.g., from MDP solvers, deep RL, or automated DAG analyzers), produce a **machine-checkable certificate** that (a) the claimed profitability bound is correct, or (b) no policy achieves revenue above a certified ceiling. The certificate must rigorously account for:

1. **Truncation error** — MDPs with unbounded state spaces (chain lengths) are truncated at some horizon; the certificate must bound the tail contribution.
2. **Continuous-state error** — difficulty-adjustment models and DAG protocols induce continuous state variables; the certificate must bound discretization/representation error.
3. **Optimality gap** — if the policy is learned rather than exactly solved, certify the gap between the learned policy's revenue and the true supremum over all policies.

Formal target: for a mining MDP M with parameterized adversary of hash power α and network propagation γ, output a checkable artifact C(α, γ) and a proof (in a proof assistant or verified checker) of a statement of the form "expected relative revenue of the certified policy is at least r" and/or "no policy achieves more than r + ε," with ε explicitly decomposed into truncation, discretization, and learning terms.

Ambiguity note: the census does not fix the certificate formalism (Lean/Coq/Isabelle proof, or a verified numerical certificate with a trusted checker). Both are acceptable readings; the essential requirement is independent machine checkability.

## Why does it matter?

Selfish-mining profitability numbers underpin security claims for Bitcoin-style and DAG-based protocols; today these numbers come from unverifiable numerical pipelines, so protocol designers and auditors must trust the solver. A certificate framework would let a paper deploying a new consensus rule ship a checkable "no profitable attack below hash power α" artifact — a requirement for high-assurance deployment and regulatory scrutiny. It also connects to certified RL: blockchain MDPs are a clean, high-impact testbed where verification errors have direct monetary interpretation.

## What's already known?

- Bar-Zur, Eyal & Tamar (2020) introduced efficient MDP formulations for selfish mining, including **Markov chains with time-varying state** to handle difficulty adjustment; they compute profit thresholds numerically but with no formal certificates — truncation of the (infinite) state space is heuristic, and difficulty states are handled by discretizing a continuous variable without explicit error bounds.
- Classical analyses (Eyal & Sirer 2013; Sapirshtein, Sompolinsky, Zohar 2016) establish optimal selfish mining via finite truncated MDPs; the truncation is justified by monotonicity/structure arguments that are pen-and-paper, not mechanized.
- Keller (2025) (verify details) automates selfish-mining search for **DAG-based PoW protocols**, apparently by automated state-space construction and analysis — but again numerically, without proof-producing output.
- Adjacent fields supply tools but no integration: **probabilistic model checking** (PRISM etc.) can verify reward bounds on finite MDPs but struggles with unbounded and continuous state; **certified RL** (e.g., verified value iteration, interval bounds on Bellman operators) gives certified upper/lower bounds on MDP values but has not been applied to mining MDPs with difficulty adjustment or DAG structure.

Gap: no existing work combines (i) unbounded-state truncation with explicit error, (ii) continuous difficulty variables with rigorous discretization bounds, and (iii) machine-checkable output.

## What would count as solving it?

Any of the following, in increasing strength:

1. **Certified value bound (finite case):** For the truncated finite MDP (fixed max chain length L, no continuous state), a mechanically checked proof that the computed optimal relative revenue is within δ of the value of the truncated MDP — e.g., via verified value iteration with rational/interval arithmetic.
2. **Truncation error bound:** A proved bound on |V_L − V_∞| as a function of L (using drift/contractivity of the chain-length dynamics), so finite certificates lift to the unbounded MDP.
3. **Continuous-state certificate:** A rigorous discretization error bound for the difficulty/time state (e.g., Lipschitz value function argument), checked mechanically.
4. **Full system:** An end-to-end pipeline: protocol description (chain or DAG) → automated MDP extraction → learned/computed policy → proof-producing certificate in Lean/Coq/Isabelle or via a verified checker, with all error terms explicit, demonstrated on Bitcoin-style selfish mining and at least one DAG protocol.

A referee would accept: reproducible artifact + machine-checked theorem statement covering optimality gap and all truncation/discretization terms, with case study reproducing known thresholds (e.g., the ~0.25 selfish-mining threshold for γ = 0.5, optimal-revenue curves of Sapirshtein et al.).

## How to start

1. Read Bar-Zur–Eyal–Tamar (arXiv:2007.05614), especially the difficulty-adjustment (time-varying) state construction — this is where the continuous-state error originates and where prior work is least rigorous.
2. Read Sapirshtein et al. 2016 (verify exact venue) for the truncation-by-monotonicity argument that you will need to mechanize; it is the crux of lifting finite to infinite state.
3. Prototype the "easy" case first: finite truncated MDP, exact rational arithmetic, verified value iteration (or interval arithmetic in Isabelle/Lean with existing probability libraries); certify the Eyal–Sirer revenue formula as a sanity milestone.
4. Main obstacle: the tail/truncation argument relies on structural monotonicity claims about optimal policies that are true empirically but subtle to state and prove; expect to find the cleanest invariant is about the value function's growth in chain length, not the policy itself.

## Key references

- Bar-Zur, R., Eyal, I., Tamar, A. (2020). *Efficient MDP Analysis for Selfish-Mining in Blockchains*. arXiv:2007.05614. — Formulates selfish mining with difficulty adjustment as efficiently solvable MDPs with time-varying states; the numerical baseline whose claims need certifying. https://arxiv.org/abs/2007.05614
- Keller (2025). *Automated Selfish Mining Analysis for DAG-Based PoW Consensus Protocols*. arXiv:2501.10888. — Automates attack discovery/state-space analysis for DAG protocols; supplies the modern automated-analysis hook this problem asks to make proof-producing. https://arxiv.org/abs/2501.10888
- Sapirshtein, A., Sompolinsky, Y., Zohar, A. (2016). *Optimal Selfish Mining Strategies in Bitcoin*. FC 2016. — Establishes optimal selfish-mining revenue via truncated MDPs; source of the truncation-by-L argument to mechanize. (verify)
- Eyal, I., Sirer, E.G. (2014). *Majority is not Enough: Bitcoin Mining is Vulnerable*. FC 2014. — The original selfish-mining attack and revenue analysis; sanity target for certification. (verify)
- Certified RL / probabilistic model checking literature (PRISM, verified value iteration) — background toolset; no blockchain-specific results yet.

## Warnings

- The "revenue" objective differs subtly across papers (relative revenue vs. absolute reward rate, with/without difficulty feedback); fix the definition before proving anything, as error bounds are not transferable between them.
- Truncation is not always benign: with difficulty adjustment, naive truncation can flip the sign of profitability conclusions — don't assume finite-MDP convergence without a drift argument.
- Beware of unverifiable "verified" pipelines: if the checker itself is not trusted/verified, the certificate is only as good as the checker.
- The 2025 DAG reference is recent; independently verify its exact scope and methods before relying on it.

---
*`BCGT-000234` · flagged for review*

## Acceptance guidance

Any of the following, in increasing strength:

1. **Certified value bound (finite case):** For the truncated finite MDP (fixed max chain length L, no continuous state), a mechanically checked proof that the computed optimal relative revenue is within δ of the value of the truncated MDP — e.g., via verified value iteration with rational/interval arithmetic.
2. **Truncation error bound:** A proved bound on |V_L − V_∞| as a function of L (using drift/contractivity of the chain-length dynamics), so finite certificates lift to the unbounded MDP.
3. **Continuous-state certificate:** A rigorous discretization error bound for the difficulty/time state (e.g., Lipschitz value function argument), checked mechanically.
4. **Full system:** An end-to-end pipeline: protocol description (chain or DAG) → automated MDP extraction → learned/computed policy → proof-producing certificate in Lean/Coq/Isabelle or via a verified checker, with all error terms explicit, demonstrated on Bitcoin-style selfish mining and at least one DAG protocol.

A referee would accept: reproducible artifact + machine-checked theorem statement covering optimality gap and all truncation/discretization terms, with case study reproducing known thresholds (e.g., the ~0.25 selfish-mining threshold for γ = 0.5, optimal-revenue curves of Sapirshtein et al.).

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/6a0a3628-ea6f-4f94-84f3-965cf1b337bd?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000234.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000234)
