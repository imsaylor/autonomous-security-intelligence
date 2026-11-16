# Discovery Pass 1 — 2026-11-16

First systematic GitHub search for benchmark material, security
tooling, and audit-report sources. All figures below are GitHub API
results from 2026-11-16, not remembered numbers.

## Benchmark targets (verified)

| Repository | Stars | Forks | Updated | Notes |
| ---------- | ----- | ----- | ------- | ----- |
| OpenZeppelin/damn-vulnerable-defi | 759 | 1,486 | 2026-11-12 | DeFi challenge set; the benchmark for economic/logic bugs |
| OpenZeppelin/ethernaut | 2,348 | 785 | 2026-11-16 | Classic smart-contract challenge set |
| nicolasgarcia214/damn-vulnerable-defi-foundry | 515 | 190 | 2026-11-11 | Foundry port of DVD; matches the local toolchain |
| ciaranmcveigh5/ethernaut-x-foundry | 278 | 72 | 2026-10-23 | Foundry port of Ethernaut |
| StErMi/foundry-ethernaut | 109 | 38 | 2026-06-16 | Alternative Foundry port |
| StErMi/forge-damn-vulnerable-defi | 68 | 13 | 2026-06-02 | Alternative Foundry port |

Assessment: the Foundry ports matter because the working toolchain is
Foundry. Before running any of them, the code must be inspected
(setup scripts, dependencies) and installation approved — per the
repository's no-untrusted-code rule. The official OpenZeppelin
repositories are the canonical sources; ports are conveniences to be
verified, not trusted.

## Static analysis tooling (verified)

| Repository | Stars | Updated | Notes |
| ---------- | ----- | ------- | ----- |
| crytic/slither | 6,420 | 2026-11-15 | Static analyzer by Trail of Bits; Python. Not installed yet. |
| trailofbits/slither-mcp | 99 | 2026-11-09 | MCP server for Slither; relevant to AI-assisted review. |
| pessimistic-io/slitherin | 377 | 2026-08-21 | Additional Slither detectors. |

Assessment: Slither is the first tool to integrate — highest
adoption, maintained, directly applicable to the existing lab
contracts as a first test target (run it against the vulnerable vault
and check that it flags the known reentrancy).

## Audit-report collections (negative result)

Searches for Code4rena and Sherlock audit-report collections on
GitHub returned only small, unmaintained mirrors (0–23 stars, stale).
No authoritative collection was found. Conclusion: competition audit
reports must be read from the platforms' own sites (code4rena.com,
sherlock.xyz, cantina.xyz), not from GitHub mirrors. That research is
still to do; nothing here claims it was done.

## Next searches

- GitHub Security Advisories for Solidity/EVM ecosystems.
- GitHub Topics: `smart-contract-security`, `solidity-security`.
- Auditor repositories: Trail of Bits, ConsenSys Diligence,
  OpenZeppelin audit reports, Spearbit.
- Fuzzing/invariant tooling: Echidna, Medusa, Halmos — verify
  repositories and maintenance status before any install.
