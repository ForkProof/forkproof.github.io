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
- Responsible disclosure. Vulnerability details are never published without the project's consent.
- A review is not a guarantee of security; it reports what was examined and found.

## Public reports

| Date | Project | Scope | Findings | Report |
|------|---------|-------|----------|--------|
| Oct 2026 | OpenZeppelin `ERC7786OpenBridge` (community-contracts `0361935`) | N-of-M cross-chain bridge: message counting, sender authentication, replay, re-entrancy | No exploitable vulnerability; 4 design observations | [Repository and case study](https://github.com/ForkProof/oz-erc7786-fuzz) |

## Contact

hello@forkproof.com

---

ForkProof is operated by Modulout LLC.
