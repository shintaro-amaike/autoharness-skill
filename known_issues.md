# AutoHarness Claude Code Skill — Known Issues

## Overview

Five structural gaps identified between the current SKILL.md design and
a faithful implementation of the AutoHarness refinement loop
(arXiv:2603.03329).

---

## Issue 1 — No Scoring: Pass/Fail Only

**Severity:** High

**Problem:**  
`harness_check.py` returns only `{"passed": true/false, "errors": [...]}`.
There is no quantitative measure of *how close* the harness is to
convergence, making it impossible to compare candidates or decide when
to stop.

**Required fix:**  
Extend the output to include:

```json
{
  "heuristic_score": 0.85,
  "illegal_pass_rate": 0.0,
  "false_positives": [...],
  "false_negatives": [...],
  "passed": true
}
```

`heuristic_score` should follow the paper's formula:
- Any illegal code passes → `H = 0.0`
- All illegal code blocked → `H = 1.0 - (false_negative_rate × 0.5)`

---

## Issue 2 — No Convergence Criterion

**Severity:** High

**Problem:**  
The update loop has no termination condition. `/autoharness-update` can
be called indefinitely with no definition of "good enough," which means
the harness never formally converges and may oscillate.

**Required fix:**  
Add an explicit convergence check in the SKILL.md:

```
Converged when BOTH conditions hold:
  - illegal_pass_rate == 0.0
  - heuristic_score   >= 0.9

Hard cap: 10 update iterations maximum.
If not converged by iteration 10, adopt the candidate with the highest
heuristic_score observed so far.
```

---

## Issue 3 — Update Trigger Is Human-Dependent

**Severity:** Medium

**Problem:**  
The primary trigger for `/autoharness-update` is the user manually
typing the command. This breaks the autonomous refinement loop central
to the AutoHarness paper, where the synthesizer reacts to environment
feedback without human intervention.

**Required fix:**  
Define automatic trigger conditions in priority order:

```
1. illegal_pass_rate > 0 after any harness_check run  → trigger immediately
2. User says "that's wrong" / "not what I wanted" / "we use X here"
3. Two or more consecutive type errors / test failures / lint errors
4. User explicitly types /autoharness-update
```

Conditions 1–3 should cause Claude Code to invoke the update without
waiting for explicit user input.

---

## Issue 4 — False Positives and False Negatives Not Classified

**Severity:** Medium

**Problem:**  
The current update flow does not distinguish between:
- **False positives** — illegal code changes that passed the harness
- **False negatives** — legal code changes that were incorrectly blocked

Without this classification, the refinement prompt cannot target the
right sections of `harness.md` (tighten rules vs. relax over-constraints),
so updates may fix one error while introducing another.

**Required fix:**  
Classify every harness failure before writing the update prompt:

| Failure type | Root cause | Target section in harness.md |
|---|---|---|
| False positive | Missing prohibition rule | Add to "Forbidden Patterns" |
| False negative | Over-constraint | Relax rule or add exception |

Pass both lists explicitly into the refinement prompt template.

---

## Issue 5 — No Maximum Iteration Guard

**Severity:** Low

**Problem:**  
There is no upper bound on update iterations. In pathological cases
(e.g., conflicting lint rules, flaky tests) the loop could run
indefinitely or produce an ever-growing `harness.md` that becomes
unreadable.

**Required fix:**  
Add a hard cap and a fallback strategy:

```
max_iterations: 10

On reaching the cap without convergence:
  - Adopt the harness with the highest heuristic_score seen so far.
  - Append a warning block to harness.md listing unresolved issues.
  - Report to the user: "Harness did not fully converge.
    Run /autoharness-update again after addressing the listed issues."
```

---

## Summary Table

| # | Issue | Severity | Affected component |
|---|---|---|---|
| 1 | Pass/fail only — no H score | High | `harness_check.py` output |
| 2 | No convergence criterion | High | SKILL.md update loop |
| 3 | Update trigger is human-dependent | Medium | `/autoharness-update` firing conditions |
| 4 | FP / FN not classified | Medium | Refinement prompt template |
| 5 | No maximum iteration guard | Low | SKILL.md update loop |

Addressing issues 1 and 2 first will have the highest impact, as all
other improvements depend on having a reliable score and a defined
stopping condition.