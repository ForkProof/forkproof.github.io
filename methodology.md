---
title: "How ForkProof tests smart contracts: invariant fuzzing on mainnet forks"
description: "Our method for finding real, reproducible smart-contract vulnerabilities — reverse-mapping to payable impact, deriving the solvency invariant independently of the code, and adversarial fuzzing against real on-chain state with no mocks."
---

# How we test: deriving the invariant, not trusting the code

Most security reviews read the code, see a guard that looks defensive, and move on. That is how real bugs survive audits — the guard was reviewed, not *broken*. ForkProof works the other way around: we treat every protection as a hypothesis to falsify with a test, against the real deployed system, until either an exploit reproduces or the invariant provably holds.

This post is how we actually do that. It is also why a clean *no finding* from us is worth something: it means we tried to break the thing with executable tests and real money on a forked chain, not that we skimmed the source.

## 1. Reverse-map to a payable impact first

We don't start from "this code looks suspicious." We start from the impact that would matter — theft of user funds, permanent freezing, protocol insolvency, bad debt socialised to lenders — and work backwards to the code path that could cause it. An in-scope code defect is not the same as an in-scope, payable, *reachable* impact. Picking the impact first keeps the whole review pointed at something that matters to the protocol's users and its treasury, instead of a pile of informational nits.

## 2. Derive the solvency condition independently

Before we read the project's own guard, we write down — on our own terms — the condition the system must satisfy to stay solvent. *After any sequence of user actions, the debt a position owes must never exceed what its collateral can cover at the liquidation threshold.* Then we compare that independent condition against what the code actually enforces.

This matters because testing a guard against *its own formula* only proves the code is self-consistent — it reproduces the author's assumptions instead of checking reality. We've seen protections that perfectly enforce the wrong invariant. The only way to catch that is to derive the right one yourself and diff.

## 3. Fork the real chain — no mocks

Every test runs against a pinned mainnet (or L2) fork, using the real deployed contracts, the real price oracles, and real token balances. We do not mock the oracle, we do not invent a friendly price feed, and we do not stand up a toy version of the protocol. A constant-price mock oracle can "prove" an exploit that cannot exist on-chain, and it can hide one that does. Realistic external state is non-negotiable — it is the difference between a finding a project can reproduce and a false alarm that wastes everyone's time.

## 4. Attack with adversarial and differential fuzzing

Normal-flow tests show the system working. Bugs live in the sequences nobody wrote a test for: deposit, borrow, withdraw, and re-balance interleaved in one transaction; an action taken one block before an epoch closes; the same operation run a thousand times with fuzzed amounts.

Two techniques do most of the work:

- **Independent-invariant fuzzing.** We assert our *own* solvency condition (section 2) after thousands of randomised action sequences. If the guard and reality ever disagree, the fuzzer finds the counterexample and minimises it to a reproduction.
- **Differential testing.** Run the same scenario twice — once with an adversarial action, once without — and compare the final, settled balances of every party. If an attacker's move leaves honest users even slightly worse off, that delta *is* the theft, and it shows up without us having to re-derive the "fair" outcome by hand.

We also run **positive controls**: a deliberately broken variant that the test *must* flag. A test suite that never fails on a known bug is not evidence of safety — it's evidence the test is blind.

## 5. Source-level is not the same as live-reachable

A defect in the source is only a vulnerability if an ordinary attacker can reach it on the deployed system. So we check the live configuration: which functions are enabled, which are paused, whether the pool is permissioned, whether the privileged roles an "exploit" assumes are actually obtainable. A real, reproducible defect is still worth zero if it needs keys an attacker can't get, or a feature the deployment has switched off. We verify the deployed bytecode matches the reviewed source, and we only call something confirmed when it reproduces against the real implementation — never against a mock, an invented consumer, or an unreachable deployment.

## What this looks like in practice

- **A credit-delegation protocol layered over Euler and Aave.** We derived, independently, the condition under which a borrower's position could leave its lenders with bad debt, then fuzzed deposit/borrow/withdraw sequences against it on a mainnet fork. The borrow-limit guard held across every reachable sequence; the only way to reach the one risky branch required a governance action an ordinary user cannot perform. Clean — and now we can say *why*, with a test, not a hunch.

- **A $144M on-chain private-credit vault.** We attacked the mid-epoch deposit pricing — the exact place a latecomer could, in principle, mint mispriced shares and dilute existing lenders. A differential fuzz over 160 randomised scenarios (deposit timing, size, both tranches) showed the pricing conserves value and rounds in existing holders' favour. No dilution, no theft. The protocol's design held up to real adversarial pressure, and that result is documented and reproducible.

Both of those were *no-finding* outcomes. We publish them anyway, because a tested, reproducible "this is sound, here is the method and the proof" is exactly the assurance a team wants before — and after — they go live.

## Why a documented negative is a real result

A review that reports "secure" tells you nothing. A review that says *"we modelled these invariants, ran these adversarial sequences against the live implementation, here is what we ruled out and what we did not cover"* tells you precisely how much assurance you actually bought. We'd rather hand you a documented negative than an unsupported medium.

---

**ForkProof** performs pre-launch and pre-partnership security reviews — smart contracts and key-custody — with every finding backed by a reproducible proof of concept on a local fork. If you're shipping something that holds user funds and want it stress-tested the way an attacker would: **hello@forkproof.com**.
