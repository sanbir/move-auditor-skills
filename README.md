# Move Auditor Skills

> The ultimate Claude Code skill for Sui Move smart contract security auditing — 143 attack vectors, 7 parallel agents, DeFi protocol checklists, and adversarial reasoning.

Built in the style of [pashov/skills](https://github.com/pashov/skills) (Solidity) but rebuilt from scratch for **Sui Move**. Aggregates knowledge from 10+ open-source audit, development, and security repositories.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Architecture

The same proven parallelized architecture as pashov/skills — adapted for Sui's object-centric model, capability-based access control, shared object concurrency, programmable transaction blocks, and Move's type/ability system.

### 4-Turn Orchestration

1. **Discover** — find all in-scope `.move` files, resolve reference paths
2. **Prepare** — bundle codebase + attack vectors + judging rules into per-agent files
3. **Spawn** — launch 4–6 agents in parallel (vector scan, adversarial reasoning, protocol analysis)
4. **Report** — merge, deduplicate by root cause, sort by confidence, format

### 143 Attack Vectors (5 reference files)

| File | Vectors | Focus Areas |
| --- | --- | --- |
| [attack-vectors-1](move-auditor/references/attack-vectors/attack-vectors-1.md) | 1–30 | Object model, abilities (copy/drop/store/key), capability pattern, access control, object leakage, type safety, transfer policies, OTW, Display, Kiosk, dynamic fields, version checks, package upgrades |
| [attack-vectors-2](move-auditor/references/attack-vectors/attack-vectors-2.md) | 31–60 | Shared object races, PTB flash loans, hot potato pattern, MEV, DoS via contention, pause mechanisms, clock/time, upgrade security, state migration, reinitialization, gas exhaustion, transaction ordering |
| [attack-vectors-3](move-auditor/references/attack-vectors/attack-vectors-3.md) | 61–90 | Bitwise overflow (Cetus-style), precision loss, rounding direction, first-depositor inflation, coin/balance operations, vector limits, dynamic field orphaning, fee bypass, dust attacks, BCS safety |
| [attack-vectors-4](move-auditor/references/attack-vectors/attack-vectors-4.md) | 91–120 | Oracle manipulation (staleness, confidence, spoofing), flash loan price manipulation, staking reward gaming, liquidation economics, dependency risks, ZK proof replay, TEE attestation, agent security, package upgrade authority |
| [attack-vectors-5](move-auditor/references/attack-vectors/attack-vectors-5.md) | 121–143 | Generic type confusion, entry visibility bypass, event spoofing, flash loan receipt pool binding, dependency version contagion, denylist epoch gap, constant definition errors, cast truncation, wrapping attacks, accumulator ordering — sourced from 200+ real Move audits |

### 3 Specialized Agent Types

| Agent | Mode | Model | Approach |
| --- | --- | --- | --- |
| Vector Scan (x5) | Default + Deep | Sonnet | Systematic triage of ~29 vectors each against full codebase |
| Adversarial Reasoning | Deep only | Opus | Free-form exploit hunting with Feynman questioning, state inconsistency analysis, invariant hunting |
| Sui Protocol | Deep only | Opus | Domain-specific checklists for lending, AMM, vaults, staking, bridges, governance, NFT/Kiosk, package upgrades |

### Quality Controls

- **FP Gate:** 3-check filter (concrete path, reachable entry, no existing guard)
- **Confidence Scoring:** Base 100 with deductions for privileged callers (-25), partial paths (-20), self-contained impact (-15), token/object assumptions (-10), external preconditions (-10)
- **Threshold:** Findings below 75 confidence reported without fix suggestions

---

## Install & Run

Works with **Claude Code CLI**, the **VS Code Claude extension**, and **Cursor**.

**Claude Code CLI:**

```bash
git clone https://github.com/sanbir/move-auditor-skills.git && mkdir -p ~/.claude/commands && cp -r move-auditor-skills/move-auditor ~/.claude/commands/move-auditor
```

**Cursor:**

```bash
git clone https://github.com/sanbir/move-auditor-skills.git && mkdir -p ~/.cursor/skills && cp -r move-auditor-skills/move-auditor ~/.cursor/skills/move-auditor
```

The skill is then invocable as `/move-auditor`. See the [skill README](move-auditor/README.md) for usage.

**Update to latest:** `cd` into the cloned repo and run:

```bash
git pull
# Claude Code CLI:
cp -r move-auditor/ ~/.claude/commands/move-auditor
# Cursor:
cp -r move-auditor/ ~/.cursor/skills/move-auditor
```

---

## Skills

| Skill | Description |
| --- | --- |
| [move-auditor](move-auditor/) | 143-vector security audit with 5–7 parallel agents, DeFi protocol checklists, and adversarial reasoning |

---

## What's Included

### 143 Attack Vectors (5 reference files)

Organized by attack surface:

**Object Model, Abilities & Access Control (V1–V30):** Missing capability checks, `copy`/`drop`/`store` misuse on value-bearing objects, object leakage via public functions, capability creation outside `init`, OTW validation, `public` vs `public(package)` visibility, type cosplay via generics, transfer policy bypass, Kiosk extraction without royalties, dynamic field orphaning, Display manipulation, version check missing on shared objects, struct layout breaks on upgrade.

**Shared Objects, PTBs & Concurrency (V31–V60):** Shared object race conditions (lost updates, version mismatch), DoS via transaction spam, PTB flash loan without hot potato, hot potato with `drop`/`store` (bypass), atomic price manipulation, MEV via validator ordering, missing pause mechanism, Clock timestamp granularity, missing deadline, upgrade cap not secured, state migration gaps, reinitialization via upgrade, gas exhaustion from unbounded operations.

**Arithmetic, Tokens & State Management (V61–V90):** Bitwise overflow (Cetus $223M exploit pattern), custom math library bugs, division-before-multiplication, integer underflow, unsafe casting, rounding direction exploitation, first-depositor vault inflation, round-trip profit, coin split/join accounting, Balance vs Coin confusion, vector 1000-entry limit, fee bypass, dust locking, coupled state reset, supply invariant violation.

**Oracle, DeFi & Platform-Level (V91–V120):** Stale oracle prices, confidence interval not validated, fake oracle objects, single oracle dependency, flash loan price manipulation, vault share inflation, staking reward index bugs, flash stake capture, liquidation incentive gaps, self-liquidation profit, interest during pause, bad debt not socialized, unaudited dependencies, ZK proof replay (missing nullifier), TEE attestation bypass, agent capability abuse, package upgrade authority as single point of failure.

**Advanced Sui Patterns, Type Safety & Real-World Exploits (V121–V143):** Generic type parameter not validated (#1 critical finding across 200+ Move audits), `entry` modifier overriding `public(package)` visibility, caller address as spoofable parameter, phantom type role bypass, event spoofing, object wrapping/unwrapping attacks, table key collision DoS, timestamp unit confusion (ms vs seconds), hot potato state reset (nested flash loan), missing object/UID validation, unconditional `balance::destroy_zero`, flash loan receipt pool binding (Cetus/Dexlyn pattern), denylist epoch gap for regulated coins, dependency upgrade version contagion, return values in wrong order (KriyaDEX), self-referential always-true checks, constant definition errors (Bluefin MAX_U64), cast truncation, double scaling/unit mixing, missing fee withdrawal, circular function calls, stale state from hidden external mutations, accumulator update ordering (Thala Labs).

### Protocol Checklists (75 items across 8 domains)

| Domain | Items | Key Checks |
| --- | --- | --- |
| Lending/Borrowing | 14 | Health factor includes accrued interest, liquidation incentive covers gas, bad debt socialization |
| AMM/DEX | 10 | Slippage from calldata, deadline enforced, flash swap restricted by hot potato |
| Vault/Token Accounting | 10 | First-depositor mitigated, rounding correct, share price not manipulable via direct transfer |
| Staking/Rewards | 10 | Accumulator updated before balance change, no flash stake, precision for small stakers |
| Bridge/Cross-Chain | 9 | Replay protection, rate limits, supply invariant, decimal conversion |
| Governance | 6 | Vote weight from past epoch, timelock, quorum, no double-voting |
| NFT/Kiosk | 8 | Transfer policy enforced, royalties collected, KioskOwnerCap secured, allowlist rules |
| Package Upgrade | 8 | Multi-sig UpgradeCap, version fields, struct append-only, migration function |

---

## Attributions

This skill aggregates knowledge from the following open-source repositories. We are grateful to all contributors.

### Architecture Inspiration

| Repository | Author | Contribution |
| --- | --- | --- |
| [pashov/skills](https://github.com/pashov/skills) | Pashov Audit Group | Parallelized agent orchestration pattern, FP gate, confidence scoring, vector-scan and adversarial-reasoning agent design, report formatting — adapted from Solidity/EVM to Sui/Move |

### Sui/Move Security Knowledge

| Repository | Author | Contribution |
| --- | --- | --- |
| [user-19-11/dxd-audit-kit-smartcontract](https://github.com/user-19-11/dxd-audit-kit-smartcontract) | DxD Audit | 25 documented vulnerabilities with code patterns and fixes, enterprise audit architecture, Sui-specific risk analysis (object model leaks, PTB attacks, shared object races, upgrade safety), real-world exploit case studies (Cetus $223M, Typus $3.44M, Nemo $2.4M), agent/AI security patterns |
| [nicholasgasior/Sui-MOVE-Smart-Contract-Auditing-Primer](https://github.com/nicholasgasior/Sui-MOVE-Smart-Contract-Auditing-Primer) | SlowMist | 15 audit categories (overflow, arithmetic accuracy, race conditions, access control, object management, token consumption, flash loans, permissions, upgrades, external calls, return values, DoS, gas, design logic), Sui object model security fundamentals |
| [nicholasgasior/x-engine](https://github.com/nicholasgasior/x-engine) | X-Engine | Vulnerability patterns with code examples: broken access controls (Sui + Aptos), reentrancy considerations in Move, integer overflow/underflow, Move vector limitations, Sui vs Aptos differences, gas optimization patterns |
| [nicholasgasior/ai-skills](https://github.com/nicholasgasior/ai-skills) | AI Skills | Comprehensive Sui Move development patterns: capability-based access control, object model security, upgrade safety, shared object considerations, Coin/Balance operations, Clock usage, dynamic fields, anti-patterns, network limits |
| [nicholasgasior/contracts-sui](https://github.com/nicholasgasior/contracts-sui) | OpenZeppelin | Two-step transfer wrapper pattern, delayed transfer wrapper with timelock, transfer policy enforcement — security patterns for sensitive capability transfer |
| [nicholasgasior/move-code-quality-skill](https://github.com/nicholasgasior/move-code-quality-skill) | Move Code Quality | 50+ code quality rules across 11 categories (Move 2024 Edition): package manifest, imports, structs, functions, parameter ordering, Option/Loop macros, testing patterns |
| [nicholasgasior/Move-Audit-Resources](https://github.com/nicholasgasior/Move-Audit-Resources) | Move Audit Resources | Curated audit tools (MoveBit, Sui Fuzzer, Move Prover, Sui Move Analyzer), security company references (OtterSec, Zellic, MoveBit, SharkTeam), research links (Zellic "Billion Dollar Move Bug", SharkTeam security analysis) |

| [pantheraudits/move-auditor](https://github.com/pantheraudits/move-auditor) | Panther Audits | 23 advanced attack vectors from 1141 real findings across 200+ Move audits: generic type confusion (#1 critical pattern), entry visibility bypass, flash loan receipt pool binding, dependency version contagion, denylist epoch gap, constant definition errors, cast truncation, double scaling, accumulator ordering — with real-world references (Navi, Cetus, Bluefin, Econia, KriyaDEX, Thala Labs, ThalaSwapV2, Creek Finance, Dexlyn, SuiPad, Hop Aggregator) |

### Methodology & Agents

| Repository | Author | Contribution |
| --- | --- | --- |
| [sainikethan/nemesis-auditor](https://github.com/sainikethan/nemesis-auditor) | Nemesis | Feynman questioning strategy, state inconsistency analysis — adapted for adversarial reasoning agent |
| [carni-ships/SolidSecs](https://github.com/carni-ships/SolidSecs) | SolidSecs | Protocol-specific checklist methodology (lending, AMM, vault, staking, bridge, governance) — adapted for Sui protocol agent |
| [auditmos/skills](https://github.com/auditmos/skills) | Auditmos | Lending protocol vulnerability patterns, liquidation mechanics, staking reward edge cases — adapted for DeFi attack vectors |

### Tools & Testing

| Repository | Author | Contribution |
| --- | --- | --- |
| [nicholasgasior/move-whisperer](https://github.com/nicholasgasior/move-whisperer) | Move Whisperer | Contract analysis and skill generation methodology, risk assessment framework (high/medium/low by function type) |
| [FuzzingLabs/sui-fuzzer](https://github.com/FuzzingLabs/sui-fuzzer) | FuzzingLabs | Coverage-guided fuzzing for Sui Move, stateful/stateless testing, vulnerability detector patterns |
| [nicholasgasior/ai-move-contract-auditor](https://github.com/nicholasgasior/ai-move-contract-auditor) | AI Move Auditor | LLM-powered Move contract analysis approach, structured audit report format |

---

## Real-World Exploits Referenced

| Exploit | Date | Loss | Root Cause |
| --- | --- | --- | --- |
| Cetus Protocol | 2025-05-22 | $223M | `checked_shlw` shift limit wrong (256 vs 192), enabling overflow in liquidity math |
| Typus Finance | 2025-10-15 | $3.44M | Missing authorization check on oracle `update_v2` function |
| Nemo Protocol | 2025-09-08 | $2.4M | Economic logic exploit — no post-bridge verification |

---

## License

[MIT](LICENSE) — see individual attribution repos for their respective licenses.
