# Benchmarks

Capability measurements. The point is honest before/after comparison,
not a score to show off. Finding count alone is never the metric.

## Benchmark sets

| Set | What it measures | Status |
| --- | ---------------- | ------ |
| Ethernaut (Foundry port) | Classic vulnerability identification, root-cause explanation, PoC quality | Run 1 done 2026-11-16: levels 1–10, 10/10 solved (see [ethernaut-run-1.md](ethernaut-run-1.md)); batch 2 done 2026-11-17: levels 11–15, 5/5 solved, all blind (see [batch-2-ethernaut-11-15-dvd-5-8.md](batch-2-ethernaut-11-15-dvd-5-8.md)) |
| Damn Vulnerable DeFi (Foundry port) | DeFi logic, economic exploits, chained weaknesses | Run 1 done 2026-11-16: challenges 1–4, 4/4 solved, fully blind (see [dvd-run-1.md](dvd-run-1.md)); batch 2 done 2026-11-17: challenges 5–8, 4/4 solved, fully blind (see [batch-2-ethernaut-11-15-dvd-5-8.md](batch-2-ethernaut-11-15-dvd-5-8.md)) |
| Lab contracts (own repos) | Regression check: known bugs must still be found after method changes | Baseline exists: reentrancy + access-control exploits pass locally |

## Metrics (per benchmark run)

- Bug identification accuracy (found vs present, with evidence).
- Root-cause correctness (is the stated cause the actual cause?).
- False-positive rate (claimed bugs that do not reproduce).
- PoC quality (reproducible, minimal, shows impact).
- Severity accuracy (against the challenge's or report's rating).
- Impact explanation quality (concrete loss, not vague harm).
- Time to analysis (recorded, never the headline metric).

## Baselines

Baseline run 1 completed 2026-11-16: 14/14 on the easiest material
(Ethernaut 1–10, DVD 1–4), 0 false positives, 0 missed bugs in this
batch. This measures basic identification only. Batch 2 completed
2026-11-17: 9/9 (Ethernaut 11–15, DVD 5–8), all blind, 0 false
positives. Totals now Ethernaut 15/15, DVD 8/8. Remaining harder
material (Ethernaut 16–25, DVD 9–13) is still untested; no
advanced-capability score is claimed anywhere in this repository.

## Rules

- Solutions to published challenges are studied only *after* an
  independent attempt, and the comparison is recorded (what was
  missed, and why).
- A missed vulnerability triggers the improvement cycle: cause,
  method gap, new test, skill fix, documentation.
