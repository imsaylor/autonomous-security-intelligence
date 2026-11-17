# Fuzz Testing with Medusa

## Name and description

- **Name:** Fuzz testing with Medusa
- **Description:** Coverage-guided smart contract fuzzing with Medusa
  (Trail of Bits): generate random call sequences against target contracts,
  check `property_*` functions and assertions after every sequence, and
  shrink failing sequences to minimal counterexamples with execution
  traces. Used to break invariants that unit tests and static analysis
  miss.

## Function and use

- **What it is used for:** Property/invariant testing, assertion testing,
  finding logic bugs (accounting omissions, state-machine violations) via
  counterexamples.
- **When it applies:** Self-contained contract logic with expressible
  invariants; stateful properties over call sequences.
- **When it does NOT apply:** Integration scenarios needing a live fork
  (fork mode exists in the config but is untested here); properties that
  cannot be expressed as a view function or assertion; performance or gas
  analysis.

## Sources

- Medusa repository and releases (v1.5.1, 2026-03-11):
  https://github.com/crytic/medusa
- Medusa documentation (Building Secure Contracts):
  https://secure-contracts.com/program-analysis/medusa/docs/src/
- crytic/properties (168 ready-made property tests):
  https://github.com/crytic/properties (read, not yet used)
- Evaluation artefacts: `~/workspace/skill-eval/medusa-test/` (contracts,
  configs, logs), 2026-11-17.

## How to use it

1. Install persistently (never under `/tmp`): download the official
   Linux release tarball to a file, inspect the tar listing, extract the
   single `medusa` binary to `~/workspace/tools/medusa/`, record its
   SHA-256. Version used: 1.5.1.
2. Provide compilation: a persistent venv with `crytic-compile`, and an
   explicit solc path via `--solc` (no solc-select needed):
   `~/workspace/skill-eval/venv/`, crytic-compile 0.4.2,
   solc `~/.svm/0.8.24/solc-0.8.24`.
3. In the project dir, run `medusa init`, then edit `medusa.json`:
   `coverageEnabled` MUST stay `true` (setting it `false` crashes v1.5.1
   with a nil-pointer panic in `CoverageTracer`); pass the solc path in
   `compilation.platformConfig.args`; set `slither.useSlither` to `false`
   unless Slither is installed; bound the campaign (`testLimit`, `timeout`,
   `workers` matched to available CPUs).
4. Write properties as `property_*` view functions returning `bool`, plus
   `assert()` statements where useful. Keep ONE target contract per
   directory/config: compiling a directory with two contracts sharing a
   property name made the second target report "no tests found".
5. Run: `medusa fuzz --config medusa.json --target-contracts <Name>`.
   Exit 7 with `[FAILED]` means a property broke; read the shrunk call
   sequence and execution trace before concluding anything.
6. Always run a control: the same property against correct code must pass
   (no false positive) before the fuzzer's verdict is trusted.

## Example implementation

Contracts and config live in `~/workspace/skill-eval/medusa-test/`
(`safe/` and `buggy/` subdirectories, one target each).

```solidity
// BuggyVault: withdraw() sends ETH but never updates totalWithdrawals.
function property_solvency() external view returns (bool) {
    return address(this).balance == totalDeposits - totalWithdrawals;
}
```

```text
medusa fuzz --config medusa.json --target-contracts BuggyVault
```

## Test cases

1. TP — `BuggyVault`: accounting-omission bug; `property_solvency` must fail.
2. TN — `SafeVault`: identical interface, correct accounting; the same
   property must pass.

## Test results

- 2026-11-17, Medusa 1.5.1, crytic-compile 0.4.2, solc 0.8.24:
  - TP: `[FAILED] Property Test: BuggyVault.property_solvency()`,
    5 passed / 1 failed. Shrunk counterexample: `deposit()` value 1,
    then `withdraw(1)` — property returned `false`. Reproduced in an
    isolated directory.
  - TN: `SafeVault` 6/6 passed, exit 0. No false positive on the control.
- Gotchas proven: `coverageEnabled: false` panics; multi-contract
  directories with shared property names break test discovery for the
  second target.
- Sigstore signature bundle for the release was NOT verified (no cosign
  available); recorded honestly. The binary was verified by HTTPS origin,
  tar inspection, ELF type, version output, and behavior on known cases.

## Limitations

- Small evaluation set (1 TP + 1 TN, one bug class). Broader evaluation
  on real protocol code (Relay) is pending scope confirmation.
- Fuzzing quality depends entirely on the properties written; a missing
  property means a missed bug class.
- No fork testing done here; do not assume fork mode works.
- Medusa runs its own isolated EVM; results are about the code, not about
  live-chain state.

## Version and updates

- **Version:** 0.1.0
- **Status:** Tested (narrow scope: vault accounting properties, 1 TP + 1 TN)
- **Last updated:** 2026-11-17
- **Changelog:**
  - 2026-11-17: Draft written from the isolated evaluation (TP/TN verified).
    Published to the repository 2026-11-17.
