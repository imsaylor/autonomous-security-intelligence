# Autonomous Security Intelligence

A working system for becoming a competitive Web3 security researcher —
not a collection of links. Every skill recorded here must survive the
same test: it was studied from a primary source, practiced locally,
tested against runnable code, and evaluated before it is called
usable. Reading documentation alone never counts as mastery.

## Operating rules

1. **Evidence before claims.** A skill is `Tested` only when a test run
   exists with a recorded result. Anything else is `Discovered`,
   `Studying`, or `Practiced`, and is labeled as such.
2. **Local only.** All practice runs on local Foundry projects or CTF
   code. No live target is tested without explicit written scope and
   safe harbor.
3. **No untrusted code.** External repositories, tools, and scripts are
   inspected before they are installed or run, and installation needs
   explicit approval. Nothing is executed blind.
4. **Independent analysis.** Known bug patterns are a starting point,
   not a checklist. Each analysis traces asset and control flow,
   questions developer assumptions, and tests hypotheses with a PoC.
5. **Learn from misses.** Every failed analysis and missed
   vulnerability is recorded with its cause, the method gap it exposed,
   and the fix applied. The same failure mode twice is a system bug.

## Learning cycle

Identify → Search → Study → Practice → Test → Evaluate → Improve →
Document → Integrate. A skill enters the working system only after the
Test and Evaluate steps, with results recorded in its file.

## Structure

| Directory | Contents |
| --------- | -------- |
| `skills/` | One file per capability, following `skills/TEMPLATE.md`. |
| `research/` | Study results: repositories, advisories, audit reports, write-ups. |
| `tools/` | Tools studied or integrated, with install status and verification. |
| `methodologies/` | Analysis methods: threat modeling, attack-surface mapping, reporting. |
| `vulnerability-patterns/` | Bug patterns with mechanism, detection, and PoC references. |
| `test-lab/` | Experiments and local test setups. Local-only, no live targets. |
| `knowledge-base/` | Indexed knowledge, linking to the sibling research repositories. |
| `benchmarks/` | Capability measurements: CTF results, before/after comparisons. |
| `changelog/` | Dated log of every skill and system change. |

## Skill status

| Skill | Status | Evidence |
| ----- | ------ | -------- |
| [Reentrancy analysis](skills/reentrancy-analysis.md) | Practiced | Lab exploits + fixes, 15/15 Foundry tests; OpenZeppelin source comparison |
| Access-control analysis | Practiced | Lab modules, tests green (write-up pending) |
| [Static analysis with Slither](skills/slither-static-analysis.md) | Practiced | Slither 0.11.6 evaluated in an isolated venv; lab results verified per-result (1 reentrancy TP proven, 1 access-control false negative proven); Relay output triage-only |
| Invariant / fuzz testing | Discovered | Not started |
| Signature vulnerability analysis | Discovered | Not started |
| Upgradeability bug analysis | Discovered | Not started |

Statuses: `Discovered` → `Studying` → `Practiced` → `Tested` →
`Integrated`. Only `Tested` and `Integrated` skills are used in live
bounty analysis.

## Sibling repositories

- [web3-security-research](https://github.com/imsaylor/web3-security-research) — cited study notes.
- [smart-contract-vulnerabilities](https://github.com/imsaylor/smart-contract-vulnerabilities) — vulnerable contracts, exploits, fixes.
- [solidity-learning-lab](https://github.com/imsaylor/solidity-learning-lab) — Solidity exercises with tests.
- [defi-security-research](https://github.com/imsaylor/defi-security-research) — DeFi mechanism analysis.
- [bug-bounty-reports](https://github.com/imsaylor/bug-bounty-reports) — report standard and practice reports.
- [bounty-hunting-journal](https://github.com/imsaylor/bounty-hunting-journal) — curriculum and journal.

## What this repository is not

- It is not a list of tools. A tool appears here only after it has been
  studied or integrated, with a recorded result.
- It is not a claim of expertise. Statuses above are the honest record
  of what has actually been done.
