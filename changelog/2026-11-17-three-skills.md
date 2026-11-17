# Changelog — 2026-11-17: Three skills (access control, Medusa, Halmos)

- Third skill recorded: Access-control analysis v0.1.0, status
  `Practiced`, from the Slither evaluation proof (an unauthorized caller
  redirecting fees and raising the fee to 100% in a Foundry test, flagged
  by no Slither detector) and the Relay on-chain role enumeration
  (custom role hashes, single-EOA concentration).
- Fourth skill recorded: Fuzz testing with Medusa v0.1.0, status `Tested`
  (narrow scope), from an isolated evaluation of Medusa 1.5.1: a planted
  accounting bug caught with a minimal 2-call counterexample, and the
  correct-code control passing 6/6. Two version gotchas documented
  (coverage must stay enabled; one target per directory).
- Fifth skill recorded: Symbolic testing with Halmos v0.3.3, status
  `Tested` (narrow scope), from an isolated evaluation: an overflow
  counterexample found symbolically in 0.20s, and the control property
  proved. One environment constraint documented: contract deployment in
  setUp() fails here, so the library/internal-function pattern is
  required.
- Tool inventory updated: Medusa and Halmos moved from `Discovered` to
  `Evaluated` (not yet part of the standard workflow). Slither
  reinstalled persistently after the ephemeral venv was wiped; smoke
  test green.
