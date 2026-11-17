# Changelog — 2026-11-17: Slither skill + benchmark batch 2

- Second skill recorded: Static analysis with Slither v0.1.0, status
  `Practiced` (narrow scope), from a verified install (0.11.6, hashes
  checked, isolated venv) and lab runs with per-result verification:
  1 reentrancy true positive proven by a drain test, 1 major false
  negative proven (missing access control flagged by no detector),
  transient-storage blindness documented. Relay periphery output kept
  as unverified triage only; nothing from it is presented as a finding.
- Benchmark batch 2 completed
  (`benchmarks/batch-2-ethernaut-11-15-dvd-5-8.md`): Ethernaut levels
  11–15 (5/5) and Damn Vulnerable DeFi challenges 5–8 (4/4), all solved
  blind with no references, all passing on first execution, 0 false
  positives. Totals now Ethernaut 15/15, DVD 8/8. Attempt code stays
  local; only the report is published.
- Tool inventory updated: Slither moved from `Discovered` to
  `Evaluated` (not yet part of the standard workflow).
