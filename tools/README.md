# Tools

Tools studied or integrated, with install status and verification.
A tool listed as `Discovered` has not been installed. Installation of
any external tool requires a security inspection and explicit
approval first.

## Index

| Tool | Status | Notes |
| ---- | ------ | ----- |
| Foundry (forge/cast/anvil/chisel) 1.9.0 | Integrated | Installed 2026-11-16 via foundryup; `forge build` + `forge test` verified green on two local projects (15/15 and 40/40 tests). |
| Slither (`crytic/slither`) 0.11.6 | Evaluated | Evaluated 2026-11-17 in an isolated venv (install verified, hashes recorded). Caught lab reentrancy; missed lab access control entirely; blind to transient storage. Evidence: [skills/slither-static-analysis.md](../skills/slither-static-analysis.md). Reinstalled persistently the same day after the /tmp venv was wiped; smoke test green (102 detectors). Not yet part of the standard workflow. |
| slither-mcp (`trailofbits/slither-mcp`) | Discovered | MCP server for Slither; evaluate after Slither itself works. |
| Slitherin (`pessimistic-io/slitherin`) | Discovered | Extra detectors; evaluate after Slither itself works. |
| Echidna | Discovered | Fuzzer; repository and maintenance status not yet verified. |
| Medusa (`crytic/medusa`) 1.5.1 | Evaluated | Evaluated 2026-11-17 from the official release (SHA-256 recorded); persistent install with a crytic-compile venv. Caught a planted accounting bug with a minimal 2-call counterexample; passed the correct-code control 6/6. Gotchas: coverage must stay enabled (v1.5.1 panic), one target per directory. Evidence: [skills/medusa-fuzzing.md](../skills/medusa-fuzzing.md). Not yet part of the standard workflow. |
| Halmos (`a16z/halmos`) 0.3.3 | Evaluated | Evaluated 2026-11-17 in the persistent venv (z3-solver). Found an overflow counterexample symbolically in 0.20s; proved the control property. Constraint: contract deployment in setUp() fails in this environment — library/internal-function pattern required. Evidence: [skills/halmos-symbolic-testing.md](../skills/halmos-symbolic-testing.md). Not yet part of the standard workflow. |
| Mythril | Discovered | Symbolic execution; maintenance status unclear — verify before use. |

## Integration rule

A tool moves to `Integrated` only after: (1) its source/origin was
checked, (2) it was installed with approval, (3) it ran successfully
on a known target (a lab contract with a known bug), and (4) the
result was recorded. Until then it is a candidate, not a capability.
