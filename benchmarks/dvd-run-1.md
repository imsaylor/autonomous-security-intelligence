# Benchmark Run 1 — Damn Vulnerable DeFi (Foundry port), challenges 1–4

- **Date:** 2026-11-16
- **Codebase:** `nicolasgarcia214/damn-vulnerable-defi-foundry`
  (cloned 2026-11-16, inspected before running; setup needed
  `git submodule update --init --recursive --depth 1` because
  forge-std's nested ds-test submodule was missing on a shallow
  init — full-history clone of the large OpenZeppelin repo stalled
  and was replaced by a shallow clone).
- **Toolchain:** Foundry 1.9.0.
- **Protocol:** stricter than the Ethernaut run. Only the challenge
  contracts in `src/Contracts/` were read. Each attempt used an
  independently built setup (own token deployment, own funding, own
  assertions) — no repo test file was read at any point, so these 4
  attempts are fully blind with respect to the included solutions.

## Results: 4/4 solved

| # | Challenge | Root cause identified | Exploit | Result | Gas |
| - | --------- | --------------------- | ------- | ------ | --- |
| 1 | Unstoppable | `flashLoan()` asserts `poolBalance == actual balance`, but only `depositTokens()` updates `poolBalance` | Direct 1-wei token transfer to the pool bricks all flash loans (DoS) | PASS | 89,648 |
| 2 | Naive Receiver | Anyone can trigger a flash loan naming any borrower; the borrower pays the fixed 1-ETH fee | Ten zero-amount loans drain the receiver's 10 ETH in fees | PASS | 410,978 |
| 3 | Truster | `flashLoan()` executes an arbitrary caller-chosen call from the pool | Make the pool `approve()` the attacker for its full balance, then `transferFrom` it out | PASS | 125,347 |
| 4 | Side Entrance | The balance check only verifies pool ETH is restored; `deposit()` converts borrowed ETH into withdrawable credit | Deposit the borrowed ETH inside `execute()`, then `withdraw()` | PASS | 379,396 |

## Integrity notes

- **Blind status:** full. No solution file was read before or during
  these attempts. Setups, exploits, and assertions are original.
- **Compile fixes:** 0. All 4 tests compiled and passed on the first
  execution run.
- **False positives:** 0. Each test asserted the drained/bricked end
  state directly.
- **Scope:** the first 4 (easiest) challenges. Challenges 5–13
  (Selfie, Puppet, Backdoor, Climber, etc.) involve governance,
  oracles, proxies, and multicall patterns and are untested.

## Attempt code

Kept locally at
`~/workspace/benchmarks/dvd-foundry/test/my-attempts/My*.t.sol`
(working copies of the cloned benchmark repos; not pushed to GitHub,
to avoid publishing challenge solutions as original work).
