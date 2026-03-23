# Move Auditor

A security agent for **Sui Move** packages — findings in minutes, not weeks.

Attribution: architecture and audit workflow lineage from [pashov/skills](https://github.com/pashov/skills), adapted for Sui Move.

Built for:

- **Move developers** who want fast feedback before merging changes
- **Security researchers** who need a first-pass sweep over object flows and authority boundaries
- **Auditors** who want broad vector coverage before deeper manual reasoning

It is not a substitute for a full audit. It is the fast pass you should run before you trust a package.

## Demo

_Portrayed below: running the skill in a terminal workflow_

![Running move-auditor in terminal](../static/skill_pag.gif)

## Usage

```bash
# Scan the full repo (default)
/move-auditor

# DEEP mode — adds Sui protocol analysis (opus)
/move-auditor deep

# Review specific file(s)
/move-auditor sources/vault.move sources/pool.move

# Write report to a markdown file
/move-auditor --file-output
```

## Architecture (v3)

8 parallel hacking agents (sonnet), each with a specialized methodology:

| Agent | Focus |
|-------|-------|
| Vector Scan | All attack vectors from the vector bundle |
| Math Precision | Integer arithmetic, rounding, precision, decimals |
| Access Control | Capabilities, abilities, visibility, ownership |
| Economic Security | Value flows, oracles, PTB flash loans, incentives |
| Execution Trace | PTB composition, state transitions, encoding |
| Invariant | Conservation laws, state couplings, round-trips |
| Periphery | Utility modules, math libraries, base modules |
| First Principles | Assumption extraction and violation |

DEEP mode adds a 9th agent (opus) for Sui protocol-specific checklist analysis (lending, AMM, vault, staking, bridge, governance, NFT/kiosk, upgrades).

## Coverage

- **143 attack vectors** mapped to Move and Sui-specific bugs
- **8 specialized hacking agents** for parallel analysis
- **4-gate validation** (refutation, reachability, trigger, impact) with lead promotion
- **DEEP mode** for DeFi protocol checklist analysis

## What It Looks For

- leaked or forgeable capabilities and witness misuse
- shared-object access races and missing invariants across PTBs
- unsafe dynamic-field writes and upgrade paths
- coin, balance, and treasury accounting drift
- kiosk / NFT policy bypasses
- stale oracle reads and object-version assumptions
- package-init / re-init / migration mistakes
- integer precision loss and wrong rounding direction
- cross-module assumption chains

## Tips

- **Point it at the hot modules first.** `sources/` files that own shared objects, admin caps, vault balances, or upgrade state are where the highest-value bugs usually live.
- **Use `deep` for lending, AMMs, bridges, BTCfi, and upgrade-heavy packages.** Relational bugs across capabilities, objects, and PTBs need the extra reasoning pass.
- **Run more than once.** LLM output is non-deterministic — each run can surface different vulnerabilities. Two or three passes over the same code often catch things a single pass misses.
