# Benchmark Run 1 — Ethernaut (Foundry port), levels 1–10

- **Date:** 2026-11-16
- **Codebase:** `ciaranmcveigh5/ethernaut-x-foundry` (cloned 2026-11-16,
  inspected before running; setup: `git submodule update --init`,
  `forge build`). Levels solved through the repo's own
  `Ethernaut`/`Level` framework: each attempt had to pass the level's
  `validateInstance` check, which is the objective ground truth.
- **Toolchain:** Foundry 1.9.0, solc 0.8.30 (auto-selected).
- **Protocol:** read the challenge contract and its factory (the win
  condition) only; write an independent exploit test; run it; then
  compare with the included solution. Solution test files were not
  read before attempting.

## Results: 10/10 solved

| # | Level | Root cause identified | Exploit | Result | Gas |
| - | ----- | --------------------- | ------- | ------ | --- |
| 1 | Fallback | Fallback grants ownership to any contributor, regardless of amount | Tiny contribution + 1-wei fallback call, then withdraw | PASS | 2,589,460 |
| 2 | Fallout | Misspelled "constructor" is a callable public function | Call `Fal1out()` | PASS | 2,327,581 |
| 3 | CoinFlip | Randomness from `blockhash(block.number - 1)` is predictable | Attacker contract recomputes the flip; one flip per block via `vm.roll` | PASS | 2,698,626 |
| 4 | Telephone | `tx.origin != msg.sender` check is satisfied via any intermediate contract | Attacker contract calls `changeOwner()` | PASS | 2,018,763 |
| 5 | Token | `unchecked` transfer underflows instead of reverting | Transfer balance + 1 | PASS | 2,222,817 |
| 6 | Delegation | Fallback delegatecalls arbitrary calldata into `Delegate.pwn()` | Call `pwn()` through the fallback | PASS | 2,054,810 |
| 7 | Force | No payable function, but ether can be forced in | Constructor `selfdestruct` to the instance | PASS | 1,636,123 |
| 8 | Vault | `private` storage is publicly readable | Read slot 1 with `vm.load`, call `unlock()` | PASS | 1,839,557 |
| 9 | King | Failed payout to the previous king reverts the whole claim | Become king with a contract that cannot receive ether | PASS | 2,211,525 |
| 10 | Re-Entrancy | `withdraw()` sends ether before updating the balance | Recursive receiver drains the instance | PASS | 2,643,077 |

## Integrity notes (read before citing this run)

- **Partial exposure, levels 3–4:** while listing level directories,
  the first lines of the repo's shipped `CoinFlipHack.sol` and
  `TelephoneHack.sol` solution contracts were incidentally displayed.
  The exploits were still derived from the challenge contracts and
  implemented independently (own attacker contracts, own test flow),
  but levels 3–4 cannot be called fully blind. Levels 1–2, 5–10 were
  attempted without seeing any solution material.
- **Compile fixes:** 3 trivial type-conversion fixes in my own test
  files (payable address conversions under solc 0.8.30) were needed
  before the first run. No exploit-logic change was needed: all 10
  hypotheses were correct on first execution.
- **False positives:** 0. Every attempt asserted the level's own
  validation; no claimed bug failed to reproduce.
- **Scope:** these are the 10 easiest levels. Levels 11–25
  (Gatekeeper, PuzzleWallet, Motorbike, etc.) are substantially harder
  and untested. This run is a baseline for basic identification, not
  evidence of advanced capability.

## Solution comparison

Spot-checked the included solutions for King and CoinFlip after my
attempts: same root causes, same exploit shapes (non-receiving king
contract; same-block recomputation). No approach difference found.

## Attempt code

Kept locally at `~/workspace/benchmarks/ethernaut-x/test/My*.t.sol`
(working copies of the cloned benchmark repos; not pushed to GitHub,
to avoid publishing challenge solutions as original work).
