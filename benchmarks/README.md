# Benchmarks

Capability measurements. The point is honest before/after comparison,
not a score to show off. Finding count alone is never the metric.

## Benchmark sets

| Set | What it measures | Status |
| --- | ---------------- | ------ |
| Ethernaut (Foundry port) | Classic vulnerability identification, root-cause explanation, PoC quality | Run 1 done 2026-11-16: levels 1–10, 10/10 solved (see [ethernaut-run-1.md](ethernaut-run-1.md)) |
| Damn Vulnerable DeFi (Foundry port) | DeFi logic, economic exploits, chained weaknesses | Run 1 done 2026-11-16: challenges 1–4, 4/4 solved, fully blind (see [dvd-run-1.md](dvd-run-1.md)) |
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
batch. This measures basic identification only. Harder levels and
challenges (11–25, 5–13) are the real test and are scheduled next.
Until then, no advanced-capability score is claimed anywhere in this
repository.

## Rules

- Solutions to published challenges are studied only *after* an
  independent attempt, and the comparison is recorded (what was
  missed, and why).
- A missed vulnerability triggers the improvement cycle: cause,
  method gap, new test, skill fix, documentation.
