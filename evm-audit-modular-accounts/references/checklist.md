# Modular Smart Accounts Security Checklist (ERC-7579 / 6900 / 7484)

Every item here is non-obvious — basic AA / 4337 issues are in `evm-audit-erc4337`.

## Module Install / Uninstall Authorization

- [ ] **`installModule` callable without account-owner auth**: A module is anything that gains execution rights on the account. If `installModule` doesn't go through the account's own validator (only checked at UserOp level), a single compromised validator can install attacker modules permanently. Look for: `installModule(moduleType, module, initData)` callable by anyone, or by `address(this)` without a validator gate. [ERC-7579]

- [ ] **Self-uninstall blocked, leaving stuck malicious module**: A module's `onUninstall(bytes)` reverts, blocking removal. The account is permanently bound to it. Look for: account implementations that revert if uninstall hook reverts (rather than catching). [Rhinestone audit]

- [ ] **`onInstall(bytes initData)` trusted blindly**: Module's install hook is given attacker-controllable `initData`. If the hook does `IERC20.approve(initData_decoded.spender, max)` on the account, anyone installing the module drains it. Look for: modules whose `onInstall` performs token approvals or external state writes parametrized by initData. [ERC-7579; Solodit]

- [ ] **Install replay across accounts via signature**: Modules installed via signed UserOp where the signature doesn't include the account address allow the same signature to install on another account with the same nonce. Look for: install UserOps signed without `account` in the EIP-712 struct. [ERC-7579]

- [ ] **No registry check, or default registry is mutable**: ERC-7484 lets accounts consult a registry before installing. If the account skips the check (`useRegistry == false` by default) or trusts a registry that can be re-pointed by an admin, malicious modules slip in. Look for: install flows that don't call `IERC7484.check(module, ...)` or trust a single registry without a per-account allowlist. [ERC-7484]

- [ ] **Registry attestation revocation not honored**: A registry attestation can be revoked. Accounts cache the "trusted" decision at install time and never re-check. Look for: time-of-check vs time-of-use — module installed when trusted, later revoked, still executes. [ERC-7484]

## Validator Module Attacks

- [ ] **Validator returns success regardless of signature**: A buggy/malicious validator's `validateUserOp` returns `SIG_VALIDATION_SUCCESS` (`0`) for any input. Once installed it's a free key. Look for: validators with weak signature checks, especially "session key" validators where the key set isn't tightly scoped. [ERC-7579]

- [ ] **Session-key validator without scope binding**: Session keys are supposed to restrict to specific selectors / targets / value caps. A validator that accepts any UserOp signed by a session key is just another root key. Look for: session-key modules whose validation doesn't decode and check `callData` selector and target. [Biconomy Nexus; Pashov]

- [ ] **Validator install grants pre-existing nonce range**: 4337 v0.7 uses 192-bit nonce keys; a validator-per-key approach lets a malicious installed validator pick a key already in use, replaying old UserOps under its new selection. Look for: nonce-key namespacing without enforcing uniqueness per validator. [ERC-4337 v0.7; ERC-7579]

- [ ] **Multi-validator OR-mode degrades security to weakest**: Some accounts let multiple validators each independently authorize a UserOp. Adding one weak validator (e.g., a passkey with no replay protection) lowers security globally. Look for: `OR` composition of validators without a strict mode flag. [ERC-7579]

- [ ] **Signature uses validator address as prefix, not enforced**: 7579 signatures often pack `[validator address (20)][signature]`. If the account doesn't enforce that the prefix matches an INSTALLED validator, an attacker can pick any address whose `validateUserOp` always succeeds. Look for: signature unpacking that calls validator at the prefixed address without checking install list. [ERC-7579]

## Executor Module Attacks

- [ ] **Executor calls `execute` on the account without scoping**: Executors get `executeFromExecutor(target, value, data)`. A malicious executor can call any target on behalf of the account, including `transfer(attacker, balance)`. Look for: any executor installed without strict per-target / per-selector allowlist. [ERC-7579]

- [ ] **Executor module + Validator module collusion**: One installed executor + one weak validator = full account drain even if individually each looks limited. Look for: protocol or wizard that bundles "default" sets of modules — review them as a group. [Rhinestone]

- [ ] **Recurring-payment executor without rate limit reset**: Executor that pulls funds on a schedule (subscriptions). If the rate-limit counter isn't bounded by total spend, it accumulates infinitely after a long pause. Look for: subscription/streaming modules. [Solodit]

## Hook Ordering & Reentrancy

- [ ] **Pre/post hook order undefined for multiple hooks**: Multiple hooks installed; order of execution depends on installation order (linked list). A later-installed hook can intercept the earlier hook's writes. Look for: hooks that rely on each other's invariants without enforcing relative order. [ERC-7579]

- [ ] **Hook can revert post-execution, masking success**: Post-execution hook reverts after action already happened (e.g., funds left). Some accounts treat this as "tx failed" and the UserOp is reverted by the entry point — but state side-effects in the action's external calls already committed (e.g., token approvals to external contracts). Look for: external state changes between action and post-hook. [ERC-7579]

- [ ] **Hook installs another hook**: Hooks running with account privileges can call `installModule` to add more modules. Look for: hook code without `nonReentrant` on install paths, or installModule callable from hook context. [Pashov]

- [ ] **Hook bypassed for direct module calls**: `executeFromExecutor` may skip hooks (depending on impl). Look for: any execution path that doesn't traverse the hook list when hooks were supposed to be universal. [ERC-7579 audits]

## Fallback Handler Hijack

- [ ] **Unrestricted fallback handler swallows ERC-1271 `isValidSignature`**: Installing a fallback handler that catches the `isValidSignature(bytes32,bytes)` selector lets the handler approve arbitrary signatures, bypassing the account's validators. Look for: install flows that allow user to register fallback for any selector, including standard ERC-1271/165/777/721/1155. [ERC-7579; ERC-1271]

- [ ] **Handler shadows core account function**: A handler registered for, say, `executeUserOp` selector could intercept calls intended for the core. Look for: install paths that don't reject handlers registering reserved selectors. [ERC-7579]

- [ ] **Handler executes with account privileges via `delegatecall`**: Some implementations dispatch via delegatecall (handler runs in account's storage). A malicious handler arbitrary-writes storage. Look for: fallback dispatch using `delegatecall` to a user-installed module. [ERC-6900]

- [ ] **`receive()` routed to handler, drains via reentrancy**: A handler registered for `receive`/empty selector executes on inbound ETH. Look for: ETH-receive handlers without `nonReentrant`. [ERC-7579]

## Cross-Cutting / Standards Quirks

- [ ] **Storage collision between modules**: All modules' state lives in the account (when delegated/embedded) or in module-controlled storage. Modules using fixed-slot storage will collide across installs. Look for: modules not using ERC-7201 namespaced storage. [ERC-7201; ERC-6900]

- [ ] **`onInstall` consumes gas required by next UserOp**: 4337 entry-point pre-funds gas; if a malicious onInstall burns most of it, the next legitimate UserOp reverts OOG, possibly allowing module to gain leverage. Look for: unbounded loops in onInstall hooks. [ERC-7579]

- [ ] **Module type masks not enforced**: ERC-7579 uses a single `moduleTypeId` (validator=1, executor=2, fallback=3, hook=4). A module that claims multiple types can install once and gain ALL roles. The account must reject duplicate-type installs or enforce one-type-per-module. Look for: install flows that take a `moduleType` arg without verifying the module's self-declared `isModuleType`. [ERC-7579]

- [ ] **No event on install/uninstall**: Monitoring agents can't detect malicious module installs in real time. Look for: missing `ModuleInstalled` / `ModuleUninstalled` events. [ERC-7579 Spec]

- [ ] **Registry adapter trusts attestation timestamp not checked**: Attestations have `expirationTime`. Adapters that don't verify it install expired-attestation modules. Look for: ERC-7484 adapters omitting expiry checks. [ERC-7484]
