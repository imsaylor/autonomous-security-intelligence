# Skill: Slither Static Analysis

## Name and description

- **Name:** Slither static analysis
- **Description:** Running Slither, the Solidity static analyzer,
  on local code, verifying every result by hand or with a Foundry
  test before calling it anything, and knowing exactly where the
  tool is blind — so its output is treated as a list of candidates
  for review, never as findings.

## Function and use

- **What it is used for:** A first pass over a codebase before
  manual review: catching reentrancy patterns, arbitrary-send
  patterns, unchecked calls, and structural facts (who writes
  which state, under what authorization) that focus the manual
  hunt.
- **When it applies:** Any local Solidity project that compiles
  (Foundry projects, or single files with resolvable imports).
  Most valuable on small, self-contained code and as a triage
  aid on real codebases.
- **When it does NOT apply:** Judging access control without
  annotations (Slither has no generic unprotected-function
  detector); judging reentrancy in code with a non-standard
  `nonReentrant` guard (the check is name-based); reasoning
  about transient storage (invisible to Slither's state model);
  judging signature/permit logic beyond syntactic patterns; or
  declaring code clean because Slither was silent. Those are
  manual-review and Foundry-test jobs, not scanner jobs.

## Sources

- Slither README and detector table, crytic/slither (master),
  read 2026-11-17. https://github.com/crytic/slither
- Slither wiki, Usage and Detector-Documentation.
  https://github.com/crytic/slither/wiki/Usage
  https://github.com/crytic/slither/wiki/Detector-Documentation
- Slither 0.11.6 installed source (detector implementations):
  `detectors/reentrancy/reentrancy.py`,
  `detectors/erc/erc20/arbitrary_send_erc20.py`,
  `core/declarations/function.py` (`is_reentrant`, `is_protected`).
- Full evaluation (install verification with hashes, per-result
  verification, Relay triage listing): retained locally,
  unpublished. Key results are summarized in Test results below.

## How to use it

1. Verify before installing: confirm the PyPI project
   `slither-analyzer` (Trail of Bits, homepage
   `github.com/crytic/slither`), pin the version, and check the
   downloaded artifact's metadata and hash. Install only into
   an isolated venv, never system-wide, never with sudo.
   Version tested: 0.11.6 (crytic-compile 0.4.2, Python 3.12.3).
2. Never run Slither in place on a read-only target. `slither .`
   on a Foundry project runs `forge clean` and `forge build`,
   which writes to the project directory. Copy the project to
   /tmp first; for projects with empty submodules, use
   single-file solc mode on a copy with submodules populated.
3. Run the full detector set first and export JSON:
   `slither . --disable-color --json report.json`.
   Then narrow with `--detect` for a focused pass and
   `--exclude-informational --exclude-optimization` to cut
   noise. Keep `forge` on `PATH` for Foundry mode.
4. Run the printers, not just the detectors:
   `slither . --print vars-and-auth,entry-points`. The
   `vars-and-auth` table (state written per function, with
   `msg.sender` conditions) surfaces missing access control
   that no detector flags.
5. Verify EVERY result before reporting it: re-read the exact
   source lines, then prove it with a Foundry test (exploit
   test for a suspected vulnerability, behavior test for a
   suspected false positive) or explicit code reasoning
   recorded next to the result. Raw scanner output is never
   a finding. For real-protocol code, label unverified
   scanner output as such and hand it to manual review.
6. Check for the known blind spots on every run: a modifier
   named `nonReentrant` suppresses reentrancy detectors
   regardless of what it does; transient-storage state
   (`tstore`/`tload`) does not appear in Slither's state
   model; `protected-vars` only fires on annotated variables;
   the permit variant of `arbitrary-send-erc20` requires a
   direct `permit()` call in the same function, so permit
   logic behind a library wrapper is classified under the
   sibling detector. Single-file runs produce false
   `dead-code` results for helpers used by sibling files —
   confirm cross-file references before reporting any.
7. Re-run after every fix. New code yields new results, and
   a fix can introduce a pattern the previous run never saw.

## Example implementation

Lab modules in
[smart-contract-vulnerabilities](https://github.com/imsaylor/smart-contract-vulnerabilities):
`src/reentrancy/` (vulnerable vault, attacker, storage guard,
transient-storage guard, fixed vaults) and `src/access-control/`
(unprotected vs owner-guarded treasury), plus
[solidity-learning-lab](https://github.com/imsaylor/solidity-learning-lab)
withdraw functions. Zero external dependencies; solc 0.8.24.
Slither was run on /tmp copies of these projects, never on the
staging originals.

Read-only analysis target: Relay periphery at commit `e221935`
(`RelayApprovalProxy`, `RelayRouter`, `RelayRouter_NonTstore`,
`RelayReceiver`), analyzed via solc mode on a /tmp copy with
pinned submodules, solc 0.8.28. All Relay results are unverified
scanner output; none is presented as a finding.

## Test cases

- Lab suite re-run on the analyzed copies: `forge test`, 19/19
  pass (reentrancy 4, fixed vault 4, transient guard 4, access
  control 7) — the same suite that proves the drain, the guard
  behavior, and the access-control flaw Slither missed.
- Detector-selection check: `--detect` with 5 detectors returns
  exactly 1 result (the known reentrancy); `--detect` with 15
  security detectors on the Relay proxy unit returns exactly 1
  result (the known arbitrary-send pattern).
- Single-file solc-mode check: `VulnerableVault.sol` alone
  returns the same `reentrancy-eth` result as the project run
  (plus its informational low-level-call note).
- Printer check: `vars-and-auth` shows empty `msg.sender`
  conditions on `UnprotectedTreasury` setters vs populated
  conditions on `OwnedTreasury` setters; on the Relay router
  unit it shows zero state writes for every function,
  demonstrating the transient-storage blind spot.

## Test results

- 2026-11-17, Slither 0.11.6, Foundry 1.9.0. Lab project 1:
  9 contracts, 102 detectors, 9 results — 1 true-positive
  security finding (`reentrancy-eth` on `VulnerableVault`,
  proven by the drain test), 1 minor true positive
  (`missing-zero-check`), 1 optimization true positive
  (`immutable-states`), 6 informational non-findings
  (`assembly`, `low-level-calls`). Lab project 2: 9 contracts,
  102 detectors, 4 results — 3 informational non-findings,
  1 optimization true positive. Every one of the 13 lab
  results was verified individually.
- One major false negative, proven by test: no detector flags
  the missing access control on `UnprotectedTreasury`
  (`testAnyoneCanRedirectFeesOnVulnerable` passes — an
  arbitrary address redirects fees and sets a 100% fee).
  Cause identified in detector source: `protected-vars`
  requires `@custom:security` annotations.
- No false-positive `reentrancy-eth` on any guarded or
  checks-effects-interactions withdraw in the lab (5 functions
  correctly unflagged).
- Relay periphery (read-only, unverified): 4 compilation
  units, 82 unique src-only results after dedup — High 4,
  Medium 3, Low 25, Informational 50. No `reentrancy-eth`,
  `controlled-delegatecall`, `tx-origin`, or `unchecked-*`
  result on Relay source; the absence is explained, not
  trusted (name-based guard suppression, transient-storage
  invisibility — see Limitations). Full listing retained
  locally, unpublished.

## Limitations

- No custom detector has been written and no Slither Python
  API analysis has been built; this skill covers running and
  triaging stock Slither only.
- Triage mode, `slither-disable` comments, `@custom:security`
  annotations, `--checklist`, SARIF export, and config files
  are documented but untested. Etherscan/address mode was
  deliberately never used (live-chain fetching is out of
  scope for this program's boundaries).
- No benchmark against a labeled dataset and no comparison
  with other analyzers; detection and false-positive rates
  are measured only on the small lab set above.
- Relay results come from single-file solc runs, not a
  whole-project Foundry run (blocked by empty transitive
  submodules in the read-only clones); cross-contract
  analysis may differ, and `dead-code` results from
  single-file runs are known artifacts.
- Slither cannot reason about: transient storage as state,
  non-standard guard semantics, unannotated access control,
  design intent (permissionless-by-design vs forgotten
  modifier), or signature/EIP-712/Permit2 correctness beyond
  syntactic patterns. Manual review and Foundry tests remain
  the deciding evidence.

## Version and updates

- **Version:** 0.1.0
- **Status:** Practiced
- **Last updated:** 2026-11-17
- **Changelog:**
  - 2026-11-17: created from the verified install, the
    lab runs with per-result verification, and the
    read-only Relay periphery analysis. Status stays
    `Practiced` (narrow scope) until a custom detector is
    written and tested and a labeled benchmark is run.
