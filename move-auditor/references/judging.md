# Finding Validation

Every finding passes four sequential gates. Fail any gate -> **rejected** or **demoted** to lead. Later gates are not evaluated for failed findings.

## Gate 1 — Refutation

Construct the strongest argument that the finding is wrong. Find the guard, assertion, capability check, or ability restriction that kills the attack — quote the exact line and trace how it blocks the claimed step.

- Concrete refutation (specific guard blocks exact claimed step) -> **REJECTED** (or **DEMOTE** if code smell remains)
- Speculative refutation ("probably wouldn't happen") -> **clears**, continue

## Gate 2 — Reachability

Prove the vulnerable state exists in a live deployment. Consider shared vs owned objects — shared objects are accessible by any transaction, owned objects only by the owner.

- Structurally impossible (enforced invariant, ability restriction prevents it) -> **REJECTED**
- Requires privileged actions outside normal operation (AdminCap, UpgradeCap) -> **DEMOTE**
- Achievable through normal usage, PTB composition, or common Coin behaviors -> **clears**, continue

## Gate 3 — Trigger

Prove an unprivileged actor executes the attack.

- Only capability holders can trigger -> **DEMOTE**
- Costs exceed extraction -> **REJECTED**
- Unprivileged actor triggers profitably -> **clears**, continue

## Gate 4 — Impact

Prove material harm to an identifiable victim.

- Self-harm only -> **REJECTED**
- Dust-level, no compounding -> **DEMOTE**
- Material loss to identifiable victim -> **CONFIRMED**

## Confidence

Start at **100**, deduct: partial attack path **-20**, bounded non-compounding impact **-15**, requires specific (but achievable) state **-10**. Confidence >= 80 gets description + fix. Below 80 gets description only.

## Safe patterns (do not flag)

- Standard Move arithmetic (aborts on overflow/underflow by default)
- Capability-gated admin functions (AdminCap, OwnerCap patterns)
- Hot potato enforcement (no-ability structs consumed in same transaction)
- Object ownership isolation (owned objects inaccessible to non-owners)
- `assert!` guards that cover the claimed attack path
- Virtual shares or minimum deposit mitigating first-depositor inflation
- Consistent protocol-favoring rounding unless compounding or zero-rounding

## Lead promotion

Before finalizing leads, promote where warranted:

- **Cross-module echo.** Same root cause confirmed as FINDING in one module -> promote in every module where the identical pattern appears.
- **Multi-agent convergence.** 2+ agents flagged same area, lead was demoted (not rejected) -> promote to FINDING at confidence 75.
- **Partial-path completion.** Only weakness is incomplete trace but path is reachable and unguarded -> promote to FINDING at confidence 75, description only.

## Leads

High-signal trails for manual investigation. No confidence score, no fix — title, code smells, and what remains unverified.

## Do Not Report

Linter/compiler issues, gas micro-opts, naming, documentation. Admin/capability privileges by design. Missing events. Centralization without exploit path. Implausible preconditions (but custom Coin behavior, oracle failure, and shared object contention ARE plausible for protocols accepting arbitrary tokens or using shared objects).
