# Benchmarks

Capability measurements. The point is honest before/after comparison,
not a score to show off. Finding count alone is never the metric.

## Benchmark sets

| Set | What it measures | Status |
| --- | ---------------- | ------ |
| Ethernaut (Foundry port) | Classic vulnerability identification, root-cause explanation, PoC quality | Not started |
| Damn Vulnerable DeFi (Foundry port) | DeFi logic, economic exploits, chained weaknesses | Not started |
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

No benchmark run has been completed yet. The first Ethernaut/DVD runs
will establish the baseline, and later runs will be compared against
it. Until then, no capability score is claimed anywhere in this
repository.

## Rules

- Solutions to published challenges are studied only *after* an
  independent attempt, and the comparison is recorded (what was
  missed, and why).
- A missed vulnerability triggers the improvement cycle: cause,
  method gap, new test, skill fix, documentation.
