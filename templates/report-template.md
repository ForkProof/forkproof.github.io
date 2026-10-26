# [ID] Title

| | |
|---|---|
| Project | |
| Repository / commit | |
| Severity | Critical / High / Medium / Low / Informational |
| Status | Reproduced locally / Confirmed / Fixed in `<commit>` |
| Files | `path/File.sol:L10-L42` |

## Summary
One or two sentences: what is wrong and what it lets someone do.

## Attacker and preconditions
Who can trigger it, what they need, and any assumptions.

## Expected vs actual
The invariant that should hold, and how it breaks.

## Impact
Concrete loss or lock, with amounts or scale where known.

## Proof of concept
How to run it, plus the key assertion output.

```bash
forge test --match-path test/PoC.t.sol -vv
```

## Recommendation
The smallest fix that restores the invariant.

## References
Prior audits, known issues, related findings.
