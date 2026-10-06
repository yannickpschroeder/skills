---
name: harden
description: Hardens a system in a red-green loop — probe for weaknesses, pin them as failing cases in the test bench, audit the bench itself, fix the system until green, report. Use when the user wants to harden, stress-test or make a system more robust. Usage: /harden <system description>
argument-hint: <system description>
---

Harden the system the user describes in the argument, in a loop: **Probe → Pin red → Bench audit → Green → Report**. If the argument is missing, first ask which system is meant.

## 1. Probe — find weaknesses

Locate the system's code and its existing test bench (or establish that none exists). **Read the bench first, case by case**, and write down which behaviour each case already covers — this coverage list is the filter for everything that follows: no new test for something an existing case already reproduces.

Then look for weaknesses the bench **does not show yet**: read the code, think through edge cases, gaps in the coverage list, known bug reports. You are looking for new tests that would break the system — not variations of cases that are already green.

Keep a **ledger**: one line per weakness with a concrete scenario (input/state → wrong behaviour). A weakness without a demonstrable scenario ("could be more robust") does not go into the ledger — sharpen it or drop it. An entry whose scenario an existing case already covers is dropped too — that would be double coverage.

**Done when:** every ledger entry has a scenario a test can reproduce, and none of them is already covered by the existing bench.

## 2. Pin red — nail the weaknesses down in the bench

Extend the existing test bench with one case per ledger entry; if none exists, create one following the repo's conventions. Every case gets an explicit, checkable acceptance criterion (pass/fail or threshold) when it is created.

**Real-world data first:** build cases from real data whenever possible — existing fixtures, frozen real cases (snapshots), production or live artefacts. If real data exists for the scenario but is not yet a fixture, freeze it following the repo's conventions instead of inventing data. Synthetic inputs only when no real data is available — and then marked as synthetic in the case.

Run the full bench and record the result as the baseline. The gate: **every new case must be red.** A case that is green right away does not demonstrate its weakness — either the weakness is refuted (drop it from the ledger, note it in the report) or the test misses it (rewrite it).

**Done when:** every remaining ledger entry has a red case and the baseline of the existing cases is documented.

## 3. Bench audit — does the bench measure correctly at all?

After the baseline run, BEFORE the first system fix: check the bench itself. A fix against a broken bench optimises into the void — bench errors are fixed first, then the baseline runs again.

- **Blind spots — green although broken (false negatives):** Suspicious is every case whose assertion is lax (checks only "runs through", checks a side effect instead of the behaviour) or whose expected value was derived from the system itself instead of independently. Spot-check by mutation: deliberately and temporarily break the behaviour under test — if the case stays green, it checks nothing; sharpen the assertion.
- **False alarm — red although the system is right (false positives):** For every red case, verify the oracle first, then suspect the system: read the fixture/snapshot yourself, recompute the expected value independently. If a hand-set expected value disagrees with an automatic count, the expectation is often wrong, not the system. List corrected expectations in the report — they are NOT "weakening the test" when the old expectation was demonstrably wrong.
- **Matching logic:** If the bench maps system outputs to expectations by matching (keywords, regex, IDs), the matcher itself is a source of errors in BOTH directions — a matcher that is too loose grades foreign outputs (silent pass), one that is too strict misses correct ones (false alarm). Spot-check: read the raw system outputs of 2–3 cases and hold them against the grading — does the bench grade what actually happened?
- **Stability before judgement:** For non-deterministic systems (LLM judgements, sampling), measure a conspicuous case several times (repeats) before it counts as a system or bench error. A case that flips across runs is neither an oracle error nor a duplicate — it marks a real instability at the edge of capability: report it as "unstable", neither fix it away (overfitting spiral) nor cut it.
- **Redundancy:** Cluster cases by the behaviour they check. If several cases hit the same code path with the same acceptance criterion, keep the most meaningful one (preferably the one with real data) and drop the rest — with reasoning in the report. Careful: different real-world data on the same path is often NOT redundancy (it covers different input distributions), and two similar cases that flip differently across runs are boundary probes, not duplicates. Only cut what demonstrably measures the same thing. Note runtime/cost savings.

**Done when:** every red case either has a verified oracle (real system error), was corrected as a bench error or is marked as unstable, spot-check mutations have confirmed the critical green cases, and the baseline is re-documented after bench corrections.

## 4. Green — improve the system

Change the system until the red cases turn green. Rules:

- Touch only the system, never weaken the test: don't delete cases, don't lower thresholds set at creation. Bench corrections (oracles, redundancy) belong to step 3 — if a bench error shows up WHILE fixing, go back to step 3 including a new baseline; don't change it on the side.
- The fix must remove the weakness, not recognise the pinned test inputs — no overfitting to the cases.
- After every change, run the full bench, new and old cases.
- A case that cannot be made green stays red and goes into the report as such.

**Done when:** all new cases are green (or marked as remaining red) and no existing case has regressed against the baseline.

## 5. Report + loop question

Write the report: ledger (weakness → case → before/after), what was changed in the system, what was changed in the BENCH (corrected oracles, removed redundant cases, sharpened assertions), remaining red cases, measurements/costs if any.

Then ask the user whether another round should run — every round starts again at step 1 with fresh probing, because the fixes move the weakest spot. This is the skill's only blocking question.
