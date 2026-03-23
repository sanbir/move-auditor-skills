# Move Auditor

A security agent for **Sui Move** packages.

Attribution: this fork keeps the v2 packaging and audit workflow lineage from [pashov/skills](https://github.com/pashov/skills), adapted for Sui Move.

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
/move-auditor
/move-auditor deep
/move-auditor sources/vault.move sources/pool.move
/move-auditor --file-output
```

## Coverage

- **143 attack vectors** mapped to Move and Sui-specific bugs
- **Parallel scan agents** for fast triage on package changes
- **Deep mode** for adversarial reasoning and Sui protocol analysis

## What It Looks For

- leaked or forgeable capabilities and witness misuse
- shared-object access races and missing invariants across PTBs
- unsafe dynamic-field writes and upgrade paths
- coin, balance, and treasury accounting drift
- kiosk / NFT policy bypasses
- stale oracle reads and object-version assumptions
- package-init / re-init / migration mistakes

## Tips

- **Point it at the hot modules first.** `sources/` files that own shared objects, admin caps, vault balances, or upgrade state are where the highest-value bugs usually live.
- **Use `deep` for lending, AMMs, bridges, BTCfi, and upgrade-heavy packages.** Relational bugs across capabilities, objects, and PTBs need the extra reasoning pass.
