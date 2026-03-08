# Attack Vectors Reference — Advanced Sui Patterns, Type Safety & Real-World Exploits (5/5)

> Part 5 of 5 · Vectors 121–143 of 143 total
> Covers: generic type confusion, entry visibility bypass, event spoofing, flash loan receipt binding, dependency version contagion, denylist epoch gap, constant definition errors, cast truncation, wrapping attacks, stale external mutations
> Source: [pantheraudits/move-auditor](https://github.com/pantheraudits/move-auditor) — patterns from 1141 real findings across 200+ Move audits

---

## V121 — Generic Type Parameter Not Validated

**What:** Functions accepting generic `<T>` (especially `Coin<T>`) don't verify `T` matches the expected/whitelisted type, allowing attackers to supply worthless self-created tokens.

**Why it matters:** This is the **#1 critical vulnerability** across real Move audits. Attacker creates `Coin<FakeUSDC>`, passes it to a lending protocol that accepts `Coin<T>`, borrows real assets against worthless collateral.

**What to look for:**
- `public fun deposit<T>(coin: Coin<T>, ...)` without checking T is in a whitelist
- Lending/borrowing functions that accept any CoinType without validation
- Generic pool functions where T isn't matched against stored pool type
- Missing `assert!(type_info::type_of<T>() == stored_type)` or equivalent

**Real-world:** Navi Protocol — all lending functions lacked CoinType validation (Critical). Econia — `place_market_order` no type check (Critical).

---

## V122 — Entry Modifier Visibility Bypass

**What:** `public(package) entry` functions are callable by anyone directly via transaction — the `entry` modifier overrides `public(package)` visibility restriction.

**Why it matters:** Developer intends function to be package-internal only, but adding `entry` makes it directly callable by any transaction. Internal-only functions with `entry` become public attack surface.

**What to look for:**
- Functions marked `public(package) entry` that contain privileged operations
- Internal admin functions with `entry` modifier
- Package-scoped functions that should NOT be callable from outside

**Secure pattern:** Functions meant to be package-internal only should NOT have the `entry` modifier. Use `public(package)` without `entry`.

---

## V123 — Caller Address as Parameter (Spoofable Sender)

**What:** Functions accept a caller/sender address as a parameter instead of deriving it from `tx_context::sender(ctx)`.

**Why it matters:** When the sender address is passed as a parameter, any caller can spoof any address. The attacker passes the victim's address and performs operations on their behalf.

**What to look for:**
- `public fun action(sender: address, ...)` instead of deriving from `TxContext`
- Address parameters used for ownership checks or balance lookups
- Missing `assert!(sender == tx_context::sender(ctx))`

**Secure pattern:** Always derive caller identity from `tx_context::sender(ctx)`, never from function parameters.

---

## V124 — Phantom Type Generic Role Bypass

**What:** Role-based capability checks using generic type parameters instead of concrete types, allowing role confusion.

**Why it matters:** If `RoleCap<T>` is used for authorization where T represents the role, a user holding `RoleCap<UserRole>` can pass it where `RoleCap<AdminRole>` was expected if the function doesn't validate T.

**What to look for:**
- `fun admin_action<T>(cap: &RoleCap<T>, ...)` without checking T is AdminRole
- Generic capability patterns where the type parameter determines privilege level
- Missing concrete type assertions on role capabilities

---

## V125 — Event Spoofing for Off-Chain Systems

**What:** Events emitted with attacker-controlled data that off-chain indexers or bridges trust without verifying the emitting contract.

**Why it matters:** Any contract can emit events that look like events from a legitimate protocol. Off-chain systems that index events without verifying the source module can be deceived into crediting fake deposits, confirming fake trades, etc.

**What to look for:**
- Off-chain indexers processing events without verifying emitting package ID
- Bridge relayers that verify event data but not event origin
- Events carrying user-supplied data without sanitization

---

## V126 — Object Wrapping/Unwrapping Attacks

**What:** Objects wrapped inside other objects become inaccessible if there's no guaranteed unwrap path. Malicious contracts can trap objects permanently by wrapping without providing an unwrap mechanism.

**Why it matters:** Wrapped objects are effectively locked — they can't be accessed, transferred, or used until unwrapped. If the wrapping contract doesn't expose unwrapping, the object is permanently lost.

**What to look for:**
- Functions that wrap user objects into container structs
- Missing corresponding unwrap/extract functions
- Third-party contracts that accept and wrap objects without guaranteed return
- Objects wrapped by contracts the owner doesn't control

---

## V127 — Table Key Collision / Duplicate Key Abort

**What:** `table::add` called without checking if the key already exists, causing an abort on duplicate entries.

**Why it matters:** If a user interacts a second time, the `table::add` aborts, creating a DoS. Attacker can front-run with entries to block legitimate operations. Need `table::contains` check or insert-or-update pattern.

**What to look for:**
- `table::add(table, key, value)` without prior `table::contains(table, key)` check
- `dynamic_field::add` without existence check
- User-interaction patterns where the same key can be added twice

---

## V128 — Timestamp Unit Confusion (ms vs seconds)

**What:** `clock::timestamp_ms()` returns milliseconds but code compares against seconds constants (or vice versa), making time-based locks effectively instant or extremely long.

**Why it matters:** If a 24-hour lock is implemented as `lock_until = now + 86400` but `now` is in milliseconds, the lock is 86 seconds. If the constant is in milliseconds but `now` is in seconds, the lock is 1000 days.

**What to look for:**
- `clock::timestamp_ms(clock)` compared to constants without `_MS` suffix
- Time calculations mixing seconds and milliseconds
- Lock duration constants that seem oddly small or large
- Missing documentation on time unit expectations

**Real-world:** SuiPad — `one_day = 0` (High). Dexlyn — `DAY_SECONDS = 600` instead of 86400 (High).

---

## V129 — Hot Potato State Reset (Nested Flash Loan)

**What:** Flash loan `start` function can be called multiple times within the same PTB, resetting the initial snapshot each time, so `finish` validates against the wrong baseline.

**Why it matters:** Attacker calls `start` (snapshot balance=1000) → borrows 500 → calls `start` again (snapshot balance=500) → `finish` checks against 500, not 1000. Attacker keeps 500.

**What to look for:**
- Flash loan `start` that overwrites snapshot without checking for existing active loan
- Hot potato receipt creation without a "loan active" flag
- Multiple `start` calls composable in same PTB
- `finish` that trusts receipt parameters instead of verifying actual balance restoration

---

## V130 — Missing Object/UID Validation

**What:** Functions accepting Sui objects without validating the UID or verifying the object originates from the expected source.

**Why it matters:** Attacker creates their own instance of the same struct type with manipulated field values (fake price, fake balance, fake authority). Without UID validation, the function accepts the attacker's object.

**What to look for:**
- Functions accepting `&T` or `&mut T` where T is a shared/owned object without ID checks
- Missing `assert!(object::id(obj) == stored_id)` on critical objects
- Oracle objects accepted without verifying they come from the registered oracle

---

## V131 — Unconditional `balance::destroy_zero` on Non-Zero Balance

**What:** `balance::destroy_zero()` called on a balance that may contain a non-zero amount, permanently destroying the remaining funds.

**Why it matters:** `destroy_zero` aborts if the balance is not zero. But if wrapped in error-swallowing logic or if a related bug causes the balance to be non-zero, funds are lost or the function becomes uncallable.

**What to look for:**
- `balance::destroy_zero(remaining)` after operations that may leave dust
- Fee collection where remainder is destroyed instead of returned
- Conditional logic where one path leaves a non-zero balance but calls `destroy_zero`

**Real-world:** Creek Finance — unconditional `destroy_zero` on non-zero balances (High).

---

## V132 — Flash Loan Receipt Pool Binding

**What:** Flash loan receipts (hot potato structs) don't store the originating pool's ID, allowing receipts from one pool to repay loans from another.

**Why it matters:** Attacker borrows from Pool A (high-value), gets receipt, repays to Pool B (low-value), and the receipt is accepted because it doesn't bind to Pool A.

**What to look for:**
- Hot potato receipt struct without a `pool_id: ID` field
- `repay` function that doesn't assert `receipt.pool_id == object::id(pool)`
- Receipt structs that only store amount, not origin
- Flash loan patterns where start/finish aren't bound to the same pool instance

**Real-world:** Cetus — `repay_flash_loan` doesn't verify `order_id` (Critical). Dexlyn — `repay_flash_swap` missing pool binding (Critical).

---

## V133 — Denylist Enforcement Epoch Gap

**What:** For regulated coins using Sui's denylist, sending is blocked instantly at validator level, but receiving is only blocked at the next epoch (~24 hours).

**Why it matters:** In cross-chain scenarios: burn tokens on source chain, target address gets denied between burn and mint, mint arrives next epoch — but the epoch gap means the mint succeeds before the deny takes effect, creating stuck or misrouted funds.

**What to look for:**
- Protocols handling regulated coins (USDC, USDT on Sui) without accounting for epoch-delayed deny
- Cross-chain bridges without denylist-aware pause mechanisms
- Transfer logic that doesn't handle the ~24h enforcement gap

---

## V134 — Dependency Upgrade Version Contagion

**What:** When a Sui package dependency upgrades, the parent package's object version checks may fail permanently because objects created by the old dependency version carry the old version number.

**Why it matters:** If Protocol A (immutable) depends on Library B (upgradeable), and Library B upgrades, Protocol A's version checks on objects created pre-upgrade fail permanently. The immutable protocol is bricked by its dependency's upgrade.

**What to look for:**
- Immutable packages depending on upgradeable packages
- Object version checks that don't account for dependency upgrades
- Missing version migration logic for dependency updates
- Critical protocols without dependency pinning strategy

---

## V135 — Return Values in Wrong Order

**What:** Multi-return functions return values in incorrect order, silently corrupting all callers.

**Why it matters:** Move doesn't name return values. If `get_reserves()` returns `(reserve_a, reserve_b)` but the implementation swaps them, every caller computes with wrong values. Swaps, prices, and collateral calculations all break.

**What to look for:**
- Functions returning multiple values of the same type (e.g., two `u64` values)
- Callers destructuring multi-return in a different order than the function produces
- Missing documentation on return value ordering

**Real-world:** KriyaDEX — `get_reserves` returned values in wrong order (High).

---

## V136 — Self-Referential Validation (Always-True Checks)

**What:** Assertion compares a value to itself (`assert!(config.version == config.version)`), which always passes and validates nothing.

**Why it matters:** Looks like a security check but is a tautology. The actual intended check (comparing against an expected version or parameter) is missing.

**What to look for:**
- `assert!(x == x)` patterns
- Version checks that compare object's version against its own field instead of an expected value
- Copy-paste errors where both sides of a comparison reference the same source

**Real-world:** Hop Aggregator — version self-comparison (High).

---

## V137 — Constant Definition Errors

**What:** Hardcoded constants contain wrong values — missing digits in MAX_U64, wrong time constants (DAY_SECONDS = 600), precision constants off by orders of magnitude.

**Why it matters:** A `MAX_U64` missing one digit means overflow checks trigger too early or too late. A `SECONDS_PER_DAY = 600` (10 minutes instead of 86400) makes time-locked functions unlock 144x faster than intended.

**What to look for:**
- `MAX_U64` or `MAX_U128` — count the digits (u64 max = 18446744073709551615, 20 digits)
- Time constants: SECONDS_PER_DAY = 86400, SECONDS_PER_YEAR = 31536000
- Precision constants: 1e6, 1e9, 1e12, 1e18 — verify digit count
- Basis points: 10000 (not 1000 or 100000)

**Real-world:** Bluefin — MAX_U64 missing digit (Critical). Dexlyn — DAY_SECONDS = 600 (High). SuiPad — one_day = 0 (High).

---

## V138 — Cast Truncation (Narrowing Casts)

**What:** Narrowing casts from larger to smaller integer types (u128 → u64, u64 → u32, u64 → u8) silently truncate high bits without overflow protection.

**Why it matters:** Unlike arithmetic operations which abort on overflow in Move, casts silently truncate. A u128 value of `2^64 + 100` cast to u64 becomes `100`, potentially bypassing amount checks.

**What to look for:**
- `(value as u64)` where value is u128 and could exceed u64 range
- `(amount as u8)` where amount could exceed 255
- Missing `assert!(value <= MAX_U64)` before narrowing casts
- Financial calculations in u128 cast down to u64 for storage

---

## V139 — Double Scaling / Unit Mixing

**What:** Scaled values (e.g., amounts multiplied by an interest index) are mixed with raw/unscaled values in the same calculation.

**Why it matters:** If `scaled_balance = balance * interest_index / 1e18` and this is later compared to or added with a raw balance, the result is meaningless. Interest calculations become wildly incorrect.

**What to look for:**
- Variables named `scaled_*` or `index_*` used in arithmetic with non-scaled variables
- Interest index multiplication applied twice (double scaling)
- Deposits stored as raw amounts compared with withdrawals stored as scaled amounts

**Real-world:** AAVE v3 on Aptos — borrow index set to token decimals instead of RAY (High). ThalaSwapV2 — double-upscaling in `pay_flashloan` (Critical).

---

## V140 — Missing Fee Withdrawal Function

**What:** Protocol collects fees into a balance or counter but provides no function to extract them.

**Why it matters:** Fees accumulate permanently with no recovery path. Protocol revenue is locked forever. This is a permanent loss of value.

**What to look for:**
- Fee collection logic that increments a balance or counter
- No corresponding `withdraw_fees` or `claim_fees` function for admin
- Fee balances stored in objects without any extraction path

---

## V141 — Recursive / Circular Function Calls

**What:** Function A calls B which calls A, creating infinite recursion that exhausts gas or stack.

**Why it matters:** Circular calls cause out-of-gas aborts on any invocation, permanently bricking affected functions. If triggered by user actions, it's a DoS.

**What to look for:**
- Fee distribution functions that call swap functions that trigger fee distribution
- Callback patterns where A → B → A is possible
- Internal helper functions called from multiple paths creating cycles

**Real-world:** Baptswap — circular function calls in fee distribution (High).

---

## V142 — Stale State from Hidden External Mutations

**What:** Function reads a value (e.g., exchange rate), then makes a cross-module call that internally mutates that same value (e.g., triggers interest accrual), making the pre-read value stale.

**Why it matters:** The function operates on outdated data after the external call changed the underlying state. Interest calculations, price lookups, and balance checks become incorrect.

**What to look for:**
- Reading a value → calling an external function → using the pre-read value
- Interest-accruing protocols where any interaction triggers accrual
- Price feeds that update on access
- Missing re-read pattern after external calls

---

## V143 — Accumulator Update Ordering in Staking

**What:** Reward accumulator updated AFTER balance change instead of before, causing incorrect reward distribution.

**Why it matters:** If the accumulator is updated after a deposit, the new deposit immediately earns historical rewards. If updated after a withdrawal, the withdrawer's final rewards use the wrong accumulator value.

**What to look for:**
- `update_rewards()` called after `increase_stake()` or `decrease_stake()`
- Deposit/withdraw functions where accumulator isn't the first operation
- Missing "accumulate before mutate" pattern

**Real-world:** Thala Labs — improper accumulator update ordering (Critical, 2 findings).
