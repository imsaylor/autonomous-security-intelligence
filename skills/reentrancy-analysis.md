# Skill: Reentrancy Analysis

## Name and description

- **Name:** Reentrancy analysis
- **Description:** Identifying, proving, and fixing reentrant call
  vulnerabilities in Solidity contracts: tracing where an external
  call happens before state updates, building a working exploit
  against a local copy, and verifying that a fix (checks-effects-
  interactions, a guard, or both) closes the hole without breaking
  legitimate behavior.

## Function and use

- **What it is used for:** Auditing withdrawal, payout, and callback
  flows in single contracts.
- **When it applies:** Any function that calls an untrusted address
  (or a token contract) before finishing its own state changes.
- **When it does NOT apply:** Cross-contract and cross-function
  reentrancy, read-only reentrancy, and reentrancy through token
  callbacks (ERC-777 hooks, ERC-721 receivers) — not yet studied.
  Those are separate skills to build.

## Sources

- Smart Contract Weakness Classification Registry, SWC-107
  (Reentrancy). https://swcregistry.io/docs/SWC-107
- Solidity documentation, Security Considerations (v0.8.37).
- U.S. SEC, Report of Investigation: The DAO, Release No. 81207
  (July 25, 2017).
- OpenZeppelin Contracts source at commit `7df9390d`:
  `ReentrancyGuard`, `ReentrancyGuardTransient`, `Ownable` — compared
  against the lab versions in
  [openzeppelin-guards-compared.md](https://github.com/imsaylor/web3-security-research/blob/main/notes/openzeppelin-guards-compared.md).

## How to use it

1. Map the contract's external calls: every `.call`, `.transfer`,
   `.send`, token transfer, and call into another contract.
2. For each call, check the order: are all balance/allowance/state
   updates that the call depends on finished *before* the call?
3. If not, write the attacker's contract: a fallback/receive that
   re-enters the same function, draining more than the attacker's
   recorded balance.
4. Run the exploit as a Foundry test against the vulnerable contract
   and record the drained amount.
5. Apply the fix (checks-effects-interactions first; guard as
   defense-in-depth), re-run the exploit test, and confirm it fails
   while legitimate flows still pass.

## Example implementation

Lab modules in
[smart-contract-vulnerabilities](https://github.com/imsaylor/smart-contract-vulnerabilities):
`src/reentrancy/` — a vulnerable vault, an attacker contract, a
hand-written guard, and a fixed vault, with tests. Zero external
dependencies; the mechanism is readable in isolation.

## Test cases

- `test/reentrancy/` in the lab repository: exploit drains the vault;
  guard blocks re-entry; fixed vault resists the same attacker;
  legitimate deposits and withdrawals still work.

## Test results

- 2026-11-16, Foundry 1.9.0, `forge test`: 15/15 tests pass across
  the reentrancy and access-control modules (verified by re-running
  the suite, not by trusting the author's word).
- No benchmark against external challenge sets yet (see
  `benchmarks/`). Status stays `Practiced` until then.

## Limitations

- Single-contract scope only. Cross-contract, cross-function, and
  read-only reentrancy are untested.
- No transient-storage guard module yet (studied in source form only).
- No experience applying this to a real protocol codebase; lab
  contracts are small and purpose-built.
- False-positive risk on code review is unmeasured — no labeled
  dataset has been run through this skill yet.

## Version and updates

- **Version:** 0.1.0
- **Status:** Practiced
- **Last updated:** 2026-11-16
- **Changelog:**
  - 2026-11-16: created from the verified lab work and the
    OpenZeppelin source comparison.
