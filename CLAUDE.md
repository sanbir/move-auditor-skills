# CLAUDE.md

Instructions for Claude when contributing to this repository.

## What This Repo Is

A library of Claude AI skills focused on **Sui Move security auditing**.

This repo is not a generic Move coding helper. Contributions should improve vulnerability detection, attack-surface coverage, or audit workflow quality for Move packages.

## Structure

```
move-auditor/ # Security review of Sui Move packages while you develop.
CLAUDE.md     # This file (read by Claude Code)
```

## Rules

- Keep guidance specific to Move and Sui object semantics.
- Prefer capability, shared-object, PTB, upgrade, and accounting risks over generic blockchain advice.
- No fabricated examples - outputs must reflect real model responses.
- No secrets, API keys, or personal data.
