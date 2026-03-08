---
name: move-auditor
description: Security audit of Sui Move contracts while you develop. Trigger on "audit", "check this contract", "review for security". Modes - default (full repo), DEEP (+ adversarial reasoning + protocol analysis), or a specific filename.
---

# Sui Move Smart Contract Security Audit

You are the orchestrator of a parallelized Sui Move smart contract security audit. Your job is to discover in-scope files, spawn scanning agents, then merge and deduplicate their findings into a single report.

## Mode Selection

**Exclude pattern** (applies to all modes): skip directories `tests/`, `test/`, `build/`, `examples/`, `node_modules/` and files matching `*_test.move`, `*_tests.move`, `test_*.move`.

- **Default** (no arguments): scan all `.move` files in the sources directory using the exclude pattern. Use Bash `find` (not Glob) to discover files.
- **deep**: same scope as default, but also spawns the adversarial reasoning agent (Agent 5) and the Sui protocol analysis agent (Agent 6). Use for thorough reviews. Slower and more costly.
- **`$filename ...`**: scan the specified file(s) only.

**Flags:**

- `--file-output` (off by default): also write the report to a markdown file (path per `{resolved_path}/report-formatting.md`). Without this flag, output goes to the terminal only. Never write a report file unless the user explicitly passes `--file-output`.

## Version Check

After printing the banner, run two parallel tool calls: (a) Read the local `VERSION` file from the same directory as this skill, (b) Bash `curl -sf https://raw.githubusercontent.com/sanbir/move-auditor-skills/main/move-auditor/VERSION`. If the remote fetch succeeds and the versions differ, print:

> ⚠️ You are not using the latest version. Please upgrade for best security coverage. See https://github.com/sanbir/move-auditor-skills#install--run

Then continue normally. If the fetch fails (offline, timeout), skip silently.

## Orchestration

**Turn 1 — Discover.** Print the banner, then in the same message make parallel tool calls: (a) Bash `find` for in-scope `.move` files per mode selection, (b) Glob for `**/references/attack-vectors/attack-vectors-1.md` and extract the `references/` directory path (two levels up). Use this resolved path as `{resolved_path}` for all subsequent references.

**Turn 2 — Prepare.** In a single message, make three parallel tool calls: (a) Read `{resolved_path}/agents/vector-scan-agent.md`, (b) Read `{resolved_path}/report-formatting.md`, (c) Bash: create five per-agent bundle files (`/tmp/audit-agent-{1,2,3,4,5}-bundle.md`) in a **single command** — each concatenates **all** in-scope `.move` files (with `### path` headers and fenced code blocks), then `{resolved_path}/judging.md`, then `{resolved_path}/report-formatting.md`, then `{resolved_path}/attack-vectors/attack-vectors-N.md`; print line counts. Every agent receives the full codebase — only the attack-vectors file differs per agent. Do NOT read or inline any file content into agent prompts — the bundle files replace that entirely.

**Turn 3 — Spawn.** In a single message, spawn all agents as parallel foreground Agent tool calls (do NOT use `run_in_background`). Always spawn Agents 1–5. Only spawn Agents 6 and 7 when the mode is **DEEP**.

- **Agents 1–5** (vector scanning) — spawn with `model: "sonnet"`. Each agent prompt must contain the full text of `vector-scan-agent.md` (read in Turn 2, paste into every prompt). After the instructions, add: `Your bundle file is /tmp/audit-agent-N-bundle.md (XXXX lines).` (substitute the real line count).
- **Agent 6** (adversarial reasoning, DEEP only) — spawn with `model: "opus"`. Receives the in-scope `.move` file paths and the instruction: your reference directory is `{resolved_path}`. Read `{resolved_path}/agents/adversarial-reasoning-agent.md` for your full instructions.
- **Agent 7** (Sui protocol analysis, DEEP only) — spawn with `model: "opus"`. Receives the in-scope `.move` file paths and the instruction: your reference directory is `{resolved_path}`. Read `{resolved_path}/agents/sui-protocol-agent.md` for your full instructions.

**Turn 4 — Report.** Merge all agent results: deduplicate by root cause (keep the higher-confidence version), sort by confidence highest-first, re-number sequentially, and insert the **Below Confidence Threshold** separator row. Print findings directly — do not re-draft or re-describe them. Use report-formatting.md (read in Turn 2) for the scope table and output structure. If `--file-output` is set, write the report to a file (path per report-formatting.md) and print the path.

## Banner

Before doing anything else, print this exactly:

```

███╗   ███╗ ██████╗ ██╗   ██╗███████╗     █████╗ ██╗   ██╗██████╗ ██╗████████╗ ██████╗ ██████╗
████╗ ████║██╔═══██╗██║   ██║██╔════╝    ██╔══██╗██║   ██║██╔══██╗██║╚══██╔══╝██╔═══██╗██╔══██╗
██╔████╔██║██║   ██║██║   ██║█████╗      ███████║██║   ██║██║  ██║██║   ██║   ██║   ██║██████╔╝
██║╚██╔╝██║██║   ██║╚██╗ ██╔╝██╔══╝      ██╔══██║██║   ██║██║  ██║██║   ██║   ██║   ██║██╔══██╗
██║ ╚═╝ ██║╚██████╔╝ ╚████╔╝ ███████╗    ██║  ██║╚██████╔╝██████╔╝██║   ██║   ╚██████╔╝██║  ██║
╚═╝     ╚═╝ ╚═════╝   ╚═══╝  ╚══════╝    ╚═╝  ╚═╝ ╚═════╝ ╚═════╝ ╚═╝   ╚═╝    ╚═════╝ ╚═╝  ╚═╝

```
