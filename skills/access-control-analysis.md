# Access-Control Analysis

## Name and description

- **Name:** Access-control analysis
- **Description:** The capability to determine who is allowed to call each
  privileged function in a smart contract system, to find functions whose
  authorization is missing, wrong, or overly broad, and to prove the impact
  with an executable test from an unauthorized address. It covers owner-only
  functions, role systems (Ownable, AccessControl, custom roles), and the
  concentration of privileged roles in a single key.

## Function and use

- **What it is used for:** Finding missing access control, incorrect role
  checks, privilege-escalation paths, dangerous role-admin chains, and
  ownership concentration before they become findings.
- **When it applies:** Any contract with state-changing or fund-moving
  functions, role/owner systems, upgrade or admin functions, and any
  on-chain system whose privileged keys can be enumerated.
- **When it does NOT apply:** Proving economic exploits, off-chain signer
  security, or governance-process safety. It answers "who can call this",
  not "should this exist".

## Sources

- OpenZeppelin `Ownable.sol` v5.0.0, commit `7df9390d` (read 2026-11-16).
- Relay settlement + periphery role/ownership code, revisions
  `b2f3e0e` (relay-settlement) and `e221935` (relay-periphery), read
  2026-11-17; see `relay-deep-study-2026-11-17.md` (local).
- `slither-evaluation-2026-11-17.md` (local, unpublished): the
  `UnprotectedTreasury` false-negative proof.
- `relay-audits-and-onchain-2026-11-17.md` (local): on-chain role
  enumeration results, including the custom role hashes and the single-EOA
  concentration fact.

## How to use it

1. Enumerate every state-changing, fund-moving, and configuration function.
2. For each, record its guard: modifier, `require(msg.sender == ...)`,
   role check, or none. "None" is a candidate, not yet a finding.
3. Map the role system: role identifiers, who holds each role in code
   (constructor/defaults), and who administers each role (role admins,
   admin-of-admin chains).
4. Enumerate role holders on-chain with read-only calls (`cast call`
   `hasRole` / `owner`), using the exact role hashes from the deployed
   code — do not assume `DEFAULT_ADMIN_ROLE` (0x00) is used; Relay uses a
   custom `ADMIN_ROLE` hash.
5. For every unguarded or suspicious function, write a Foundry test that
   calls it from an unauthorized address and asserts the privileged effect
   happened. A finding exists only when the test passes.
6. Check concentration: one key holding several roles (admin + operator +
   ownership) is a documented fact to report accurately, not an assumed
   compromise.
7. Cross-check with a second method: Slither's `vars-and-auth` printer
   surfaces state variables written without an authorization pattern, but
   no Slither detector flags missing access control (proven false
   negative, 2026-11-17) — the manual test is the deciding evidence.

## Example implementation

Minimal reconstruction of the pattern proven in the Slither evaluation lab
(2026-11-17). Runnable as written with Foundry.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract UnprotectedTreasury {
    address public feeRecipient;
    uint256 public feeBps;

    constructor() {
        feeRecipient = msg.sender;
        feeBps = 50;
    }

    // BUG: no access control on either setter.
    function setFeeRecipient(address to) external {
        feeRecipient = to;
    }

    function setFee(uint256 bps) external {
        feeBps = bps;
    }
}
```

```solidity
// Foundry test: anyone can redirect fees and raise the fee to 100%.
function testAnyoneCanRedirectFees() public {
    address attacker = address(0xBEEF);
    vm.prank(attacker);
    treasury.setFeeRecipient(attacker);
    vm.prank(attacker);
    treasury.setFee(10000); // 100%
    assertEq(treasury.feeRecipient(), attacker);
    assertEq(treasury.feeBps(), 10000);
}
```

On-chain enumeration pattern (read-only, Relay Chain example):

```text
cast call <HUB> "hasRole(bytes32,address)" <ADMIN_ROLE_HASH> <ADDRESS> --rpc-url <relay-chain-rpc>
```

## Test cases

1. `UnprotectedTreasury` exploit test (Slither evaluation lab, 2026-11-17):
   unauthorized caller redirects fees and sets fee to 100%.
2. Relay role-holder enumeration via `cast call` (2026-11-17): custom
   `ADMIN_ROLE` hash `0xa49807...`, `OPERATOR_ROLE` hash `0x976670...`,
   Oracle `ORACLE_ROLE` held by a 2-of-5 multisig, and one EOA holding
   Hub ADMIN+OPERATOR, both Oracles' ADMIN, and ownership of Allocator,
   oracle multisig, and the canonical Ethereum Depository.
3. Slither `vars-and-auth` printer run over the lab contract: anomaly
   visible in the printer, zero detector findings.

## Test results

- 2026-11-17, Foundry 1.9.0: exploit test passed — the unauthorized call
  changed both privileged values. The vulnerability is real and proven.
- 2026-11-17, Slither 0.11.6: 0 detectors flagged the missing access
  control. Documented false negative; the skill does not rely on Slither
  for this class.
- 2026-11-17, `cast` read-only calls: role facts above verified live.
  One enumeration gap remains: the Hub `EDITOR_ROLE` holder was not found
  among the candidates checked; full log enumeration failed.

## Limitations

- Manual enumeration does not scale to very large codebases without
  tooling support; the method is thorough but slow.
- On-chain role enumeration is only as complete as the candidate set and
  the logs available; absence from the checked set is not proof of
  absence.
- Cannot assess off-chain signer security, key management, or governance
  process quality.
- Slither is blind to this vulnerability class (proven); do not treat a
  clean Slither run as evidence of sound access control.

## Version and updates

- **Version:** 0.1.0
- **Status:** Practiced
- **Last updated:** 2026-11-17
- **Changelog:**
  - 2026-11-17: Draft written from the Slither evaluation proof and the
    Relay on-chain role analysis. Status Practiced. Published to the
    repository 2026-11-17.
