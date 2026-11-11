---
title: "ForkProof — smart contract & key-custody security reviews"
description: "Pre-launch and pre-partnership smart-contract security reviews. Every finding ships with a reproducible proof of concept on a local fork."
---

# ForkProof

Smart contract and key-custody security reviews. Every finding ships with a reproducible proof of concept on a local fork.

## What we review

- **Key custody and operations**: who holds upgrade and admin keys, hot keys vs multisig, signing services, secret handling, deploy and emergency paths. The area most audits leave out, and where many of the largest losses started.
- **Smart contracts**: Solidity (EVM) and Rust (Solana / Anchor), including cross-chain messaging and bridge integrations.
- **Pre-partnership due diligence**: an independent view before you integrate with, invest in or list a protocol.
- **Audit preparation**: clear the obvious issues before a full audit so it costs less.

## How we work

- Source review plus local reproduction: Foundry and Anchor tests on local or forked chains. No testing against live systems without written permission.
- Each finding is labelled by what was shown: hypothesis, reproduced locally, or confirmed.
- Responsible disclosure. We don't publish vulnerability details without the project's consent — except for a live but unreachable or abandoned deployment, where we may publish a redacted advisory (no exploit, no proof of concept) after a documented attempt to notify the operator.
- A review is not a guarantee of security; it reports what was examined and found.

## Methodology

[How we test: deriving the invariant, not trusting the code](/methodology) — reverse-mapping to payable impact, deriving the solvency condition independently of the code, and adversarial fuzzing against real on-chain state with no mocks.

## Public reports

| Date | Project | Scope | Findings | Report |
|------|---------|-------|----------|--------|
| Oct 2026 | OpenZeppelin `ERC7786OpenBridge` (community-contracts `0361935`) | N-of-M cross-chain bridge: message counting, sender authentication, replay, re-entrancy | No exploitable vulnerability; 4 design observations | [Repository and case study](https://github.com/ForkProof/oz-erc7786-fuzz) |
| Nov 2026 | BidCoin (Arbitrum penny-auction) | Live staking / treasury contracts: access control on reward accounting | High: unauthorized treasury drain (redacted; disclosed to operator) | [Advisory](https://github.com/ForkProof/forkproof.github.io/blob/main/bidcoin-advisory.md) |

## Contact

hello@forkproof.com
