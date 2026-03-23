# Shared Scan Rules

## Reading

Your bundle has two sections:

1. **Core source** (inline) — read in parallel chunks (offset + limit), compute offsets from the line count in your prompt.
2. **Peripheral file manifest** — file paths under `# Peripheral Files (read on demand)`. Read only those relevant to your specialty.

When matching function names, check `fun name`, `public fun name`, `public(package) fun name`, and `public entry fun name` (Move visibility conventions).

## Cross-module patterns

When you find a bug in one module, **weaponize that pattern across every other module in the bundle.** Search by function name AND by code pattern. Finding a Balance accounting mismatch in `module_a::withdraw` means you check every other module's `withdraw` — missing a repeat instance is an audit failure.

After scanning: escalate every finding to its worst exploitable variant (DoS may hide fund theft). Then revisit every function where you found something and attack the other branches.

## Do not report

Admin-only functions doing admin things (capability-gated operations by design). Standard Move safety (abort on arithmetic overflow). Self-harm-only bugs. "AdminCap holder can rug" without a concrete mechanism.

## Output

Return structured blocks only — no preamble, no narration. Exception: vector scan agent outputs its classification block first.

FINDINGs have concrete, unguarded, exploitable attack paths. LEADs have real code smells with partial paths — default to LEAD over dropping.

**Every FINDING must have a `proof:` field** — concrete values, traces, or state sequences from the actual code. No proof = LEAD, no exceptions.

**One vulnerability per item.** Same root cause = one item. Different fixes needed = separate items.

```
FINDING | module: Name | function: func | bug_class: kebab-tag | group_key: Module | function | bug-class
path: caller -> function -> state change -> impact
proof: concrete values/trace demonstrating the bug
description: one sentence
fix: one-sentence suggestion

LEAD | module: Name | function: func | bug_class: kebab-tag | group_key: Module | function | bug-class
code_smells: what you found
description: one sentence explaining trail and what remains unverified
```

The `group_key` enables deduplication: `ModuleName | functionName | bug_class`. Agents may add custom fields.
