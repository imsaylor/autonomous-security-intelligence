# Knowledge Base

An index of knowledge actually acquired, pointing at the note or
source where it lives. This directory holds no claims of its own —
it maps to evidence elsewhere.

## Study notes (sibling repositories)

- Reentrancy: mechanism, The DAO, detection, fixes (SWC-107) —
  [web3-security-research](https://github.com/imsaylor/web3-security-research/blob/main/notes/reentrancy.md)
- Access-control failures: Parity, initialization bugs, `tx.origin`
  (SWC-105/106/112/115) —
  [web3-security-research](https://github.com/imsaylor/web3-security-research/blob/main/notes/access-control-failures.md)
- OpenZeppelin guards compared against lab versions —
  [web3-security-research](https://github.com/imsaylor/web3-security-research/blob/main/notes/openzeppelin-guards-compared.md)
- AMM spot-price oracle manipulation: constant-product math, flash
  loans, TWAP mitigations —
  [defi-security-research](https://github.com/imsaylor/defi-security-research)

## Primary sources consulted

- Solidity documentation, Security Considerations (v0.8.37).
- Smart Contract Weakness Classification Registry (SWC-105, SWC-106,
  SWC-107, SWC-112, SWC-115).
- OpenZeppelin Contracts source, commit `7df9390d` (v5.x).
- EIP-1153 (transient storage), EIP-7201 (namespaced storage layout).
- U.S. SEC, Report of Investigation: The DAO (2017).
- ConsenSys Diligence, Ethereum Smart Contract Best Practices.
- Uniswap v2 Core whitepaper (2020).

## Gaps (not yet studied)

EVM internals (opcodes, gas mechanics, storage layout in depth),
signature standards (EIP-191, EIP-712, EIP-1271), proxy patterns
(EIP-1967, UUPS, Diamond), formal verification concepts, MEV
mechanics. These are queued by the priority rules in the main README.
