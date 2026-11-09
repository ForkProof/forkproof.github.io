# Security advisory — missing access control in a live BidCoin (Arbitrum) contract

**ForkProof** · Severity: High (unauthorized treasury drain) · Status: live / unfixed at time of writing

> This advisory describes the issue and its impact. It **deliberately withholds the exact vulnerable function names, the
> attack sequence, and our proof of concept**, because the contract is still live and holds user funds. The detail below is
> enough to understand and fix the issue, not to exploit it. Full technical details were provided privately to the current
> operator.

## Summary
During a review of recently-deployed Arbitrum contracts, we found a **missing-authorization** defect in a live BidCoin
staking/treasury contract. A reward-accounting entrypoint is callable by **anyone**, while its sibling function (the
equivalent privileged operation) is correctly restricted to the owner. Because of this asymmetry, an ordinary participant
can create reward entitlements that are not backed by real deposits, then withdraw real stablecoin (USD₮0) from the treasury
beyond any legitimate entitlement — draining it and leaving honest stakers' rewards unbacked.

## Affected deployment
- Chain: Arbitrum One (42161)
- BidStake proxy: `0x854695aA88D23B67037eC48E8d4b59BEd9B94ec7`
- Treasury proxy: `0x824d543F4F686EA15AABba79b058d9fDF1191d16`
- Settlement token: USD₮0 `0xFd086bC7CD5C481DCC9C85ebE478A1C0b69FCbb9`

## Root cause (class)
An **access-control modifier was omitted** on a reward-creation/accounting function; the equivalent single-item function
carries an owner-only restriction. A second treasury-side accounting function has the same missing-authorization pattern and
amplifies the effect. The net result is that unprivileged callers can create reward entitlements they should not be able to.

## Impact
- Any participant (no special role) can mint themselves unbacked reward entitlements and withdraw the treasury's real
  stablecoin balance.
- Verified against the **live** deployment using an isolated, read-only mainnet fork (no on-chain transactions were sent).
  The treasury balance is small, but the defect is real, reachable by anyone, and unfixed.
- Attack cost is negligible (a trivial stake + gas).

## Recommended fix
- Restrict the reward-**creation** / accounting entrypoints (the batch variant and the treasury-side profit function) to the
  owner / authorized role — matching the restriction already present on the sibling function.
- Validate reward entitlements against actual deposits / accrued profits before they can be claimed.
- Note: the user-facing *claim* function being publicly callable is **not** the problem and should not be restricted to the
  owner — legitimate users must be able to claim. The fix belongs at the unauthorized **creation** of entitlements.

## Disclosure
The contract's originally published contact (`admin@bidcoin.live`) is no longer deliverable; the product has since moved to a
new operator (CryptoWins). We disclosed this issue privately to the current operator and are **withholding the technical
details and proof of concept until it is fixed**. No reward was sought or received — this is responsible-disclosure outreach.

*ForkProof — smart contract & key-custody security research.*
