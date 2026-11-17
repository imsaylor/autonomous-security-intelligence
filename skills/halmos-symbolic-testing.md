# Symbolic Testing with Halmos

## Name and description

- **Name:** Symbolic testing with Halmos
- **Description:** Formal verification of Solidity properties with Halmos
  (a16z): `check_*` Foundry test functions are executed symbolically, so
  parameters stand for ALL values of their type at once; the SMT solver
  either proves the assertion or returns a concrete counterexample. Used
  for properties where random fuzzing cannot reach the failing input
  (boundary arithmetic, exact overflow conditions).

## Function and use

- **What it is used for:** Proving invariants over all inputs, finding
  boundary-condition bugs (overflow, edge values), generating exact
  counterexamples.
- **When it applies:** Properties expressible as a Foundry test with
  symbolic inputs and `assert()`; arithmetic and state-machine properties
  on small-to-medium code.
- **When it does NOT apply:** Large code with path explosion, unbounded
  loops (unrolled to a bound only), properties needing live-chain state,
  or anything requiring unsupported cheatcodes (see Limitations).

## Sources

- Halmos repository: https://github.com/a16z/halmos and
  https://github.com/a16z/halmos-cheatcodes (read 2026-11-17).
- "Symbolic Execution for Smart Contract Verification" (Halmos from first
  principles, 9 proofs):
  https://papers.lux.network/pdfs/lux-symbolic-execution-halmos-lux-symbolic-execution-halmos.pdf
- Evaluation artefacts: `~/workspace/skill-eval/halmos-test/` (contracts,
  tests, logs), 2026-11-17.

## How to use it

1. Install persistently: `pip install halmos` into the persistent venv
   `~/workspace/skill-eval/venv/` (with z3-solver). Version used: 0.3.3.
   Foundry 1.9.0 and forge-std are required; put `~/.foundry/bin` on PATH
   (it is not on the default exec PATH).
2. Write `check_*` functions: parameters are symbolic; use `vm.assume()`
   for preconditions (preferred over `bound()`); `assert()` states the
   property. Keep functions `public`; `view`/`pure` work for stateless
   checks.
3. AVOID deploying contracts in `setUp()` in this environment: it fails
   with "Unsupported cheat code" (deployCode) in Halmos 0.3.3, with and
   without forge-std inheritance (proven 2026-11-17). Prefer the
   library/internal-function pattern: put the logic under test in
   `internal` library functions and call them directly from the checks.
   Deployment inside a `check_*` function itself is untested here.
4. Run: `halmos --function check_<name>` from the Foundry project root.
   A `[FAIL]` with a `Counterexample:` block is a proven violation — read
   the concrete values and verify them by hand before reporting.
5. Always run a control: the same property shape against correct code on
   a constrained domain must pass.

## Example implementation

Artefacts in `~/workspace/skill-eval/halmos-test/`.

```solidity
// Library pattern: no deployment, no setUp.
library BuggyMathLib {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        unchecked { return a + b; }
    }
}

contract LibMathTest is Test {
    function check_lib_buggy(uint256 a, uint256 b) public pure {
        uint256 c = BuggyMathLib.add(a, b);
        assert(c >= a);
    }

    function check_lib_safe(uint256 a, uint256 b) public view {
        vm.assume(a <= type(uint256).max - b);
        uint256 c = SafeMathLib.add(a, b);
        assert(c >= a);
    }
}
```

```text
halmos --function check_lib_buggy   # expect: counterexample, exit 1
halmos --function check_lib_safe    # expect: 1 passed, exit 0
```

## Test cases

1. TP — `check_lib_buggy`: unchecked addition; `assert(c >= a)` must fail
   for overflowing inputs.
2. TN — `check_lib_safe`: checked addition on the non-overflowing domain;
   the same assertion must hold.
3. Diagnostic — `check_minimal_buggy`: contract deployment in `setUp()`
   without forge-std; documents the environment constraint.

## Test results

- 2026-11-17, Halmos 0.3.3, z3, Foundry 1.9.0, forge-std latest, solc
  0.8.30 (chosen by forge):
  - TP: `[FAIL]`, counterexample in 0.20s, 2 paths:
    `a = 0xffff...e0000000000003`, `b = 0x3ffffffffffff9` — their sum
    overflows uint256, the wrapped result violates `c >= a`. Verified by
    reading the concrete values.
  - TN: `1 passed; 0 failed`, 0.28s. `vm.assume` works as documented.
  - Diagnostic: `setUp()` deployment failed with "Unsupported cheat
    code" in both variants (with and without forge-std `Test`).
    Constraint recorded; workaround is the library pattern above.

## Limitations

- Small evaluation set (arithmetic properties only). Stateful contracts,
  loops, and multi-contract systems are untested here.
- Loops are unrolled to a bound (default 2); proofs hold only within the
  bound. Path explosion limits complex code.
- Halmos reports assertion violations, not every revert; a bug must be
  expressed as a failing `assert` to be found.
- `setUp()` contract deployment does not work in this environment
  (proven); the library pattern is the working alternative until this is
  revisited with a newer Halmos or a different setup.

## Version and updates

- **Version:** 0.1.0
- **Status:** Tested (narrow scope: arithmetic properties via libraries, 1 TP + 1 TN)
- **Last updated:** 2026-11-17
- **Changelog:**
  - 2026-11-17: Draft written from the isolated evaluation (TP/TN verified,
    setUp constraint documented). Published to the repository 2026-11-17.
