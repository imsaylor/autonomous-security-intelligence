# Benchmark Batch 2 — Ethernaut levels 11–15 + Damn Vulnerable DeFi challenges 5–8

- **Date:** 2026-11-17
- **Codebases:** `ciaranmcveigh5/ethernaut-x-foundry` and
  `nicolasgarcia214/damn-vulnerable-defi-foundry` (same working clones
  as run 1, already security-inspected; submodules verified present
  before running; only `forge test`/`forge build` executed, no other
  repo scripts).
- **Toolchain:** Foundry 1.9.0, solc 0.8.30 (ethernaut-x, auto-selected),
  solc 0.8.17 (dvd-foundry).
- **Protocol:** attempt each level/challenge from the CHALLENGE SOURCE
  ONLY first (challenge contract + factory for Ethernaut; contracts in
  `src/Contracts/` for DVD). No shipped solution, test, `*Hack.sol`, or
  level README file was opened before the corresponding attempt passed.
  No web references were consulted for any level/challenge in this batch:
  every hypothesis was derived from the challenge source and standard
  protocol knowledge. Shipped `*Hack.sol` files were read ONLY AFTER all
  Ethernaut attempts passed, for comparison (see below). No DVD solution
  file was read at any point. Attempt code stays local and is never
  pushed or published.

## Results: 9/9 solved

### Ethernaut 5/5

| # | Level | Root cause identified | Exploit | Result | Gas |
| - | ----- | --------------------- | ------- | ------ | --- |
| 11 | Elevator | `goTo()` trusts the caller's `isLastFloor()` twice and the two answers can disagree | Attacker contract answers false on the first call, true on the second | PASS | 2,244,542 |
| 12 | Privacy | `private` storage is publicly readable; `unlock()` takes `bytes16(data[2])` | Read slot 5 with `vm.load`, pass its first 16 bytes to `unlock()` | PASS | 2,321,333 |
| 13 | Gatekeeper One | Three gates: contract caller, exact `gasleft() % 8191 == 0`, key derived from `tx.origin` | Intermediate contract; brute-force forwarded gas over an 8191-wide window; key `(1 << 32) \| uint16(tx.origin)` | PASS | 16,234,153 |
| 14 | Gatekeeper Two | `extcodesize == 0` holds during construction; gate key is `keccak256(sender) ^ (2**64 - 1)` | Call `enter()` from the attacker constructor with the XOR-derived key | PASS | 2,042,942 |
| 15 | Naught Coin | `lockTokens` guards only the overridden `transfer()`; inherited `transferFrom()` is unguarded | Player approves an attacker contract, which drains the full balance via `transferFrom()` | PASS | 4,742,659 |

### Damn Vulnerable DeFi 4/4

| # | Challenge | Root cause identified | Exploit | Result | Gas |
| - | --------- | --------------------- | ------- | ------ | --- |
| 5 | The Rewarder | `deposit()` mints accounting tokens BEFORE `distributeRewards()` takes a new-round snapshot, so a flash-loaned deposit is included in the snapshot that splits the round | Flash-loan 1M DVT in a new round, deposit, withdraw, repay; capture ~all 100 reward tokens | PASS | 908,069 |
| 6 | Selfie | Voting power is balance at last snapshot and `snapshot()` is public; a flash loan grants a transient >50% majority | Flash-loan 1.5M governance tokens, snapshot, queue `drainAllFunds`, repay, execute after the 2-day delay | PASS | 1,239,375 |
| 7 | Compromised | Oracle price is the median of 3 sources; 2 compromised sources fully control it | Post price 0, buy NFT for 1 wei, post the exchange's full balance, sell the NFT back | PASS | 600,991 |
| 8 | Puppet | Collateral is priced from Uniswap V1 SPOT balances (ETH/token) with no TWAP | Dump 1,000 DVT into the thin 10 ETH / 10 DVT exchange, crashing the price; borrow all 100,000 DVT for ~19.66 ETH | PASS | 279,520 |

## Per-level notes

### 11 — Elevator (blind, no references)

Attempt: one attacker contract implementing `Building` with a call
counter (`calls > 1`). First `forge test` run passed. Validation: the
level's own `validateInstance` (`top == true`) via
`submitLevelInstance`, asserted in-test. Caveat: none; the mechanism is
exactly the double-call trust in `goTo()`.

### 12 — Privacy (blind, no references)

Attempt: storage-layout analysis from the source — slot 0 `locked`,
slot 1 `ID`, slot 2 packed `uint8`/`uint8`/`uint16`, slots 3–5
`data[0..2]` — then `vm.load` of slot 5 and `unlock(bytes16(data[2]))`.
First run passed. Validation: `locked == false` asserted plus
`submitLevelInstance`. Caveat: none; the key was read from storage, not
recomputed from the factory's `keccak256` construction.

### 13 — Gatekeeper One (blind, no references)

Attempts: a single design, correct on first execution. Key derivation
done by hand from the three `require` statements: the low 32 bits must
equal the low 16 bits of `tx.origin` (satisfying parts one and three),
and a set bit above bit 31 satisfies part two. Gas gate solved with an
in-contract brute-force loop (`50000 + i` for `i` in `0..8191`, low-level
calls, stop on success). Two harness facts had to be established from the
framework source (not from solutions): (a) the Ethernaut harness records
the test contract as player while `entrant` is set to `tx.origin`, so
the attack call is wrapped in `vm.startPrank(player, player)` to make
`tx.origin == player`; (b) the repo's minimal `Vm` cheatcode interface
lacked the two-argument prank, so standard Foundry declarations
(`prank(address,address)`, `startPrank` overloads, `stopPrank`, `addr`)
were added to the local `test/Cheatcodes.sol` helper. Validation:
`entrant == player` asserted plus `submitLevelInstance`. The 16.2M-gas
brute-force loop fits Foundry's test gas limit; no config change was
needed. Caveat: the gas loop is a test-harness-friendly approach; on a
live chain the same search would normally be done off-chain across
multiple transactions.

### 14 — Gatekeeper Two (blind, no references)

Attempt: constructor-time `enter()` call with
`key = uint64(bytes8(keccak256(abi.encodePacked(address(this))))) ^
type(uint64).max`, deployed under `vm.prank(player, player)` so
`entrant == player`. First run passed. Validation: `entrant == player`
asserted plus `submitLevelInstance`. Caveat: none.

### 15 — Naught Coin (blind, no references)

Attempt: approve + `transferFrom` from a second address. First run
passed. Validation: `balanceOf(player) == 0` asserted plus
`submitLevelInstance`. Caveat: none. The `transfer()` override itself was
never called.

### 5 — The Rewarder (blind, no references)

Setup (independent): 1M DVT flash-loan pool, one honest 100-DVT
depositor, warp 6 days into a new round. Attempt: attacker contract
implementing `receiveFlashLoan` — approve, deposit, withdraw, repay.
First run passed. Evidence: attacker received >= 99e18 reward units
(asserted `>= 99 ether`; the round mints 100e18 total and the attacker
held 1M of 1,000,100 deposits at the snapshot); flash pool balance
unchanged at 1M; rewarder still holds exactly the honest 100 DVT.
Caveat: setup numbers are my own, not the canonical challenge numbers;
the mechanism (snapshot-after-mint ordering) is what was tested.

### 6 — Selfie (blind, no references)

Setup (independent): 2M governance-token supply, 1.5M in the pool.
Attempt: attacker contract implementing `receiveTokens` — `snapshot()`,
`queueAction(pool, drainAllFunds(receiver), 0)`, repay; warp 2 days + 1
second; `executeAction(1)`. First run passed. Evidence: pool balance 0
after execution; receiver gained exactly the 1.5M pool funds.
Caveat: none. The action id (1) and the 2-day delay were read from the
governance source.

### 7 — Compromised (blind, no references)

Setup (independent): 3 oracle sources, initial median price 1 ETH,
exchange funded with 10 ETH, attacker funded with 1 ETH. The compromised-
key premise is modelled by deploying the oracle with sources whose keys
the test holds (`vm.addr`) and using 2 of them; the third source never
posts and keeps the honest price. Attempt: post 0 from 2 sources (median
0), `buyOne` for 1 wei, post the exchange balance from 2 sources (median
= balance), approve + `sellOne`. First run passed. Evidence: exchange
balance exactly 0; attacker balance >= 10.999... ETH gained. Caveat: the
"compromise" itself (key leakage) is a premise, not a vulnerability
demonstrated on-chain; what the test proves is that 2-of-3 source control
fully controls the median and drains the exchange. Median values were
asserted at each step (1 ETH -> 0 -> 10 ETH).

### 8 — Puppet (blind, no references)

Setup (independent): real vendored Uniswap V1 (factory + exchange
template artifacts in `src/build-uniswap/v1`, deployed via `deployCode`),
exchange seeded 10 ETH / 10 DVT, pool funded with 100,000 DVT, attacker
with 25 ETH + 1,000 DVT. Two infrastructure findings, both derived from
the challenge source and artifact inspection (ABIs only — no solution
files): (a) `PuppetPool` reads the exchange's NATIVE ETH balance, which
only a V1 exchange holds, so V1 — not V2 — is the correct venue;
(b) `deployCode` rejects the V2 artifact JSONs ("duplicate field
`bytecode`") but accepts the V1 artifacts. Attempt: attacker dumps 1,000
DVT via `tokenToEthSwapInput`, then borrows the full pool balance.
First run passed. Evidence (from the execution trace): honest-price
deposit requirement asserted at exactly 200,000 ETH before the dump;
quoted ETH out ~9.9 ETH for the dump; post-dump borrow executed with
value 19,664,329,888,798,200,000 wei (~19.66 ETH); pool DVT balance 0;
attacker holds all 100,000 DVT. Caveat: setup numbers are my own; the
exchange interaction uses the real V1 constant-product pricing with the
0.3% fee, not a mock.

## Integrity notes

- **Blind status:** full for all 9. No shipped solution test, `*Hack.sol`
  contract, or level/challenge README was opened before the corresponding
  attempt passed. The only files read before attempting were: challenge
  contracts, factories, the Ethernaut harness (`Ethernaut.sol`,
  `BaseLevel.sol`), my own run-1 attempt tests (for harness patterns),
  the token/helper contracts in `src/Contracts/`, and the Uniswap V1
  artifact ABIs (function signatures only, to interface with the
  protocol). No web search or external reference was used in this batch.
- **Compile fixes:** 0. All 9 attempt tests compiled and passed on the
  first execution run; no exploit-logic change was needed. (Two throwaway
  probe tests were used for Uniswap deployment infrastructure and
  deleted: a V2 `deployCode` probe that failed on artifact format, and a
  V1 probe that passed. Neither contained exploit logic.)
- **False positives:** 0. Every attempt asserted the level's own
  validation (Ethernaut) or the drained/bricked end state (DVD)
  directly.
- **Helper change:** `test/Cheatcodes.sol` in the ethernaut-x working
  clone gained standard Foundry cheatcode declarations (two-argument
  prank/startPrank, stopPrank, addr). No challenge, factory, or harness
  contract was modified. No git commits were made; nothing was pushed.
- **Regression check:** after batch 2, all 15 Ethernaut `My*` tests
  (run 1 + batch 2) and all 8 DVD `my-attempts` tests pass together
  (15/15 and 8/8, final verification run 2026-11-17).

## Solution comparison (post-attempt only)

After all Ethernaut attempts passed, the three shipped `*Hack.sol`
contracts were read and compared:

- **Elevator:** same approach — call counter returning false once, then
  true. No difference in exploit shape.
- **Gatekeeper One:** the shipped `GatekeeperOneHack` is only a
  parameterized helper (takes gate key and gas amount as arguments, plus
  a `testenter` for checking gate three); the key/gas selection lives
  elsewhere and was not read. My self-contained loop + in-contract key
  derivation was developed independently and differs structurally from
  the helper.
- **Gatekeeper Two:** same approach — constructor-time call with
  `keccak256(sender) ^ (2**64 - 1)`. Mine spells the mask as
  `type(uint64).max`; the shipped version uses `uint64(0) - 1` in an
  `unchecked` block. Same value, same exploit shape.
- **Privacy / Naught Coin:** no shipped `*Hack.sol` exists for these
  levels; no comparison was performed beyond the levels' own validation.
- **DVD:** no shipped solution file was read at any point, before or
  after attempting; the DVD attempts remain fully blind with respect to
  the included solutions, matching the stricter run-1 DVD standard.

## Attempt code

Kept locally only (working copies of the cloned benchmark repos; not
pushed to GitHub, to avoid publishing challenge solutions as original
work):

- `~/workspace/benchmarks/ethernaut-x/test/MyElevator.t.sol`,
  `MyPrivacy.t.sol`, `MyGatekeeperOne.t.sol`, `MyGatekeeperTwo.t.sol`,
  `MyNaughtCoin.t.sol`
- `~/workspace/benchmarks/dvd-foundry/test/my-attempts/MyTheRewarder.t.sol`,
  `MySelfie.t.sol`, `MyCompromised.t.sol`, `MyPuppet.t.sol`

## Overall batch assessment

9/9 solved, all blind, all on first execution. This batch exercised
harder mechanisms than run 1: stateful callback deception (Elevator),
storage-layout reading (Privacy), gas-constrained + bitwise-gated entry
(Gatekeeper One/Two), incomplete access-control coverage (Naught Coin),
snapshot-ordering + flash-loan composability (Rewarder, Selfie),
oracle-median control (Compromised), and spot-price oracle manipulation
through a real DEX (Puppet). Two honest scope limits apply. First, these
are still guided challenge contracts with known vulnerability classes;
success here measures identification and exploitation of textbook
patterns, not discovery of unknown bugs in unaudited code. Second, the
DVD setups (funding amounts, oracle sources, exchange seeding) are my
own constructions built to exercise each mechanism, not the canonical
challenge fixtures — the mechanisms were verified, not the canonical
setups. Remaining untested: Ethernaut 16–25 (Preservation, Recovery,
Magic Number, Alien Codex, Denial, Shop, Dex, Dex Two, Puzzle Wallet,
Motorbike) and DVD 9–13 (Puppet V2, Free Rider, Backdoor, Climber,
Safe Miners), which involve proxies, delegatecall storage collisions,
multicall/account-abstraction patterns, and upgrade mechanisms, and are
expected to be substantially harder.
