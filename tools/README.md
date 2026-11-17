# Tools

Tools studied or integrated, with install status and verification.
A tool listed as `Discovered` has not been installed. Installation of
any external tool requires a security inspection and explicit
approval first.

## Index

| Tool | Status | Notes |
| ---- | ------ | ----- |
| Foundry (forge/cast/anvil/chisel) 1.9.0 | Integrated | Installed 2026-11-16 via foundryup; `forge build` + `forge test` verified green on two local projects (15/15 and 40/40 tests). |
| Slither (`crytic/slither`) 0.11.6 | Evaluated | Evaluated 2026-11-17 in an isolated venv (install verified, hashes recorded). Caught lab reentrancy; missed lab access control entirely; blind to transient storage. Evidence: [skills/slither-static-analysis.md](../skills/slither-static-analysis.md). Not yet part of the standard workflow. |
| slither-mcp (`trailofbits/slither-mcp`) | Discovered | MCP server for Slither; evaluate after Slither itself works. |
| Slitherin (`pessimistic-io/slitherin`) | Discovered | Extra detectors; evaluate after Slither itself works. |
| Echidna | Discovered | Fuzzer; repository and maintenance status not yet verified. |
| Medusa | Discovered | Fuzzer; repository and maintenance status not yet verified. |
| Halmos | Discovered | Symbolic testing; repository and maintenance status not yet verified. |
| Mythril | Discovered | Symbolic execution; maintenance status unclear — verify before use. |

## Integration rule

A tool moves to `Integrated` only after: (1) its source/origin was
checked, (2) it was installed with approval, (3) it ran successfully
on a known target (a lab contract with a known bug), and (4) the
result was recorded. Until then it is a candidate, not a capability.
