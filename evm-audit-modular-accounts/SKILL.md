---
name: evm-audit-modular-accounts
description: Modular smart accounts — ERC-7579, ERC-6900, and ERC-7484 registries. Load when auditing a smart-account implementation that supports plug-in validators, executors, hooks, or fallback handlers. Covers malicious modules, install/uninstall authorization, hook ordering, registry trust, fallback-handler hijack, and signature-module bypasses.
---

# EVM Audit — Modular Smart Accounts (ERC-7579 / 6900 / 7484)

Load this when the contract is:
- An **ERC-7579** smart account (Rhinestone, Biconomy Nexus, ZeroDev, etc.)
- An **ERC-6900** modular account
- A module (validator, executor, hook, fallback handler) intended to plug into one of the above
- An **ERC-7484** trust registry / adapter
- Any AA implementation that lets the user install third-party code into their account

## Why a dedicated skill

Modular accounts split a smart account into: **validators** (decide if a UserOp is authorized), **executors** (perform actions on behalf of the account), **hooks** (run pre/post each execution), and **fallback handlers** (catch arbitrary selectors). The flexibility creates a large new attack surface:

- A malicious module installed once can persist across upgrades
- Hooks run with the account's privileges
- The "registry" the account checks before installing may itself be compromised, or the account may not check at all
- Fallback handlers can hijack any selector the core account doesn't define
- Validators that don't bind signatures to the call data being executed enable signature-replay attacks across actions

## Reference Files
- `references/checklist.md` — Full dense checklist
