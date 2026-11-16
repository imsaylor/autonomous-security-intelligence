# Test Lab

Experiments and local test setups. Rules:

1. **Local only.** Everything runs on a local Foundry project or a
   local node. No mainnet, no testnet with other people's assets, no
   live target.
2. **Inspect before running.** Cloned challenge code and setup
   scripts are read before execution. No `curl | bash`, no blind
   `npm install` on unreviewed code.
3. **Record everything.** Each experiment notes its setup, commands,
   tool versions, and outcome — pass or fail.

## Planned setups

| Setup | Purpose | Status |
| ----- | ------- | ------ |
| Ethernaut (Foundry port) | Baseline: classic vulnerability identification | Not started — code inspection pending |
| Damn Vulnerable DeFi (Foundry port) | Baseline: DeFi logic and economic exploits | Not started — code inspection pending |
| Slither vs lab contracts | First tool integration test: does Slither flag the known reentrancy? | Not started — install pending approval |

## Current lab work

The working labs live in the sibling repositories until an experiment
here needs its own setup:

- [smart-contract-vulnerabilities](https://github.com/imsaylor/smart-contract-vulnerabilities)
- [solidity-learning-lab](https://github.com/imsaylor/solidity-learning-lab)
