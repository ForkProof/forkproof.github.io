---
title: "When collateral can be burned by another loan"
description: "A ForkProof case study: a cross-loan lifecycle issue in The Gavel Protocol (Arbitrum One), reproduced, responsibly disclosed and published by Gavel as KI-4 (Low)."
date: 2026-11-24
author: ForkProof
severity: Low
---

*The Gavel Protocol · [KI-4](https://github.com/JamieFrame/The-Gavel-Protocol/blob/main/docs/known-issues.md#ki-4--a-position-nft-used-as-collateral-is-burned-when-its-own-loan-settles) (published 18 Nov 2026) · ForkProof*

In November 2026, ForkProof reported a cross-loan lifecycle issue in The Gavel Protocol, an oracle-free, auction-based lending protocol on Arbitrum One. The Gavel team reviewed and reproduced the issue, published it as KI-4 in its official known-issues documentation, and assessed the severity as Low.

At the block recorded in Gavel’s disclosure, the deployed NFTLoanProtocol had no auctions or loans (loanNonce() = 0 at Arbitrum One block 525,334,168).

## The short version

Each funded Gavel loan has two transferable ERC-721 position NFTs representing the borrower and lender positions. On settlement, the protocol burns both position NFTs. Separately, NFTLoanProtocol accepts ERC-721 tokens as collateral, including Gavel’s own position NFTs.

1. A position NFT from loan X is pledged as collateral for loan Y, and loan Y is funded. NFTLoanProtocol holds the position NFT in escrow.
2. Loan X settles, burning its position NFTs, including the token currently escrowed as loan Y’s collateral.
3. Loan Y can no longer settle. Both repayment and collateral claim attempt to transfer the burned token and revert.

As a result, loan Y’s lender cannot recover its principal through settlement, while loan Y’s borrower retains the borrowed amount. If the pledged NFT is loan X’s lender position, repayment of loan X is sent to NFTLoanProtocol and remains trapped there. If it is loan X’s borrower position, loan X can instead default and its lender can claim loan X’s collateral; loan Y remains unable to settle. These are distinct outcomes.

## Why it happens

Two behaviors interact unexpectedly:

- **Generic collateral acceptance.** NFTLoanProtocol allows Gavel’s own position-NFT contracts as collateral for auctions.
- **Position burning on settlement.** Settling loan X burns its position NFTs even if one is held by the protocol as collateral for another active loan.

The missing cross-loan invariant is that collateral held against an active loan must remain available for that loan’s own settlement. In KI-4, another loan’s lifecycle can destroy the collateral asset without resolving the loan it secures. This is a state-integrity failure rather than an ordinary change in the collateral’s market price.

## How we found it

Our review focused on the interaction between features: Gavel issues transferable loan-position NFTs and also permits NFT-backed lending. We examined what happens when a position NFT becomes collateral for a second loan and the first loan settles.

Before disclosure, we compared the scenario with the published audit findings, existing known issues and trust-assumptions documentation. We did not find this cross-loan position-as-collateral scenario documented there.

## How we tested it

1. **Foundry reproduction.** We created loan X, pledged its lender position as collateral for funded loan Y, then repaid loan X. The collateral NFT was burned; attempts to settle loan Y reverted.
2. **Positive control.** A comparable flow using a conventional ERC-721 collateral token settled successfully, isolating the problem to the position token’s separate burn lifecycle.
3. **Economic and state check.** We tracked the loan proceeds separately from repayments. Loan Y’s borrower retains the borrowed principal while its lender cannot settle; in the lender-position scenario, loan X’s repayment is trapped in NFTLoanProtocol instead of reaching the holder of loan X’s lender position. Gavel assessed this as Low severity, noting that an attacker gains nothing beyond an ordinary default on loan Y.
4. **Deployed-code check.** After matching the verified deployed source to the repository, we repeated the scenario on a local Arbitrum One fork using the deployed contracts. No transaction was broadcast to the live network.

Gavel subsequently reproduced the issue in its own test suite, including an Arbitrum One fork test, and documented additional conditions and outcomes, including the borrower-position/default case, in its KI-4 disclosure.

## Mitigation and v2

For a v1-style design, a direct preventive measure would be to reject both Gavel position-NFT contracts as collateral at all auction-creation entry points, including createAuction and createAuctionFor. More broadly, a collateralized protocol must account for assets that can cease to exist while in escrow because of an external lifecycle event.

The deployed v1 contracts are immutable. Gavel’s published guidance therefore advises lenders to inspect an auction’s collateral contract and allows integrators or curation layers to exclude the affected position-NFT collections. Gavel’s hosted interface curates which collateral is listed (per the repository); direct protocol interactions remain permissionless.

Gavel says v2 is unaffected by design: repayments are credited to claims rather than pushed to current holders, and claims and positions cannot be deposited in the vault securing a loan.

## Disclosure timeline

| Date | Milestone |
|---|---|
| 29 Oct 2026 | Issue identified and reproduced locally by ForkProof |
| 10 Nov 2026 | Privately reported to the Gavel security team |
| November 2026 | Gavel reviewed the finding; ForkProof reviewed the proposed disclosure text |
| 18 Nov 2026 | Gavel published KI-4 with ForkProof attribution |

## Acknowledgment

Thank you to Jamie Frame and the Gavel team for their careful handling of the responsible disclosure: independently reproducing the issue, documenting affected and unaffected conditions, running regression tests, and inviting review of the published text.

## About ForkProof

ForkProof investigates smart-contract and cross-system security issues, with particular attention to interactions between components and lifecycle edge cases. This case study illustrates our approach: understand the protocol’s invariants, construct a reproducible scenario, and coordinate disclosure with maintainers.

**Official source:** [The Gavel Protocol — KI-4](https://github.com/JamieFrame/The-Gavel-Protocol/blob/main/docs/known-issues.md#ki-4--a-position-nft-used-as-collateral-is-burned-when-its-own-loan-settles) · Security reviews: [hello@forkproof.com](mailto:hello@forkproof.com)
