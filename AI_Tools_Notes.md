# AI_Tools_Notes.md
## Notes on AI-Assisted Work — Campus Hiring QA Exercise

---

## AI Tool Used

**Tool:** Google Gemini (Antigravity AI Coding Assistant)
**Mode:** Interactive pair-programming session
**Date:** 2026-08-21

---

## What AI Was Used For

| Task | AI Role | Human Validation Required |
|---|---|---|
| Interpreting intentionally vague requirements | AI helped identify ambiguities and surface assumptions | Yes — I reviewed each assumption for logical correctness |
| Structuring TestStrategy.md | AI proposed the document structure; I reviewed scope decisions | Yes — In-scope/out-of-scope decisions reflect QA judgement |
| Generating test case IDs and table structure | AI generated initial table format and test case shells | Yes — Every Expected Output was manually reasoned through |
| Identifying boundary conditions | AI suggested first-day/last-day and below/at/above threshold patterns | Yes — These are standard boundary value analysis techniques I validated |
| Identifying conflict scenarios | AI generated conflict resolution test cases | Yes — The stacking vs. highest-value logic reflects deliberate design assumption |
| Generating TraceabilityMatrix | AI mapped requirements to test IDs | Yes — I cross-verified each mapping is correct |

---

## What AI Was NOT Used For

- **Actual test execution** — No code was run; this is a design exercise
- **Making business decisions** — All assumptions (A1–A8) represent my own QA reasoning
- **Explaining trade-offs** — The prioritisation (P0/P1/P2/P3) reflects my own severity assessment

---

## Validation Statement

I have reviewed every test case, assumption, and matrix row in this submission.
I can explain:
- Why any test case has the expected output it does
- The reasoning behind each assumption (e.g., why equal-value rules use precedence, not random selection)
- How conflict resolution logic works and why I chose to test it the way I did
- The boundary value analysis technique applied to quantity and date-range tests

I am prepared to walk through any test case or script during the debrief.

---

## Key Decisions Made During This Exercise

### Decision 1: Stacking as Opt-In
The requirements say "rules can stack OR highest value wins OR precedence applies."
I assumed **stacking is opt-in** (configured per rule) because allowing all rules to stack by default could result in unintended large discounts. This is the safer default.

### Decision 2: Boundary Inclusive
For quantity >= 3, I confirmed "3+ items" means **quantity = 3 triggers the discount** (not only quantity = 4+). The "+" in "3+" grammatically means "3 and above" which is inclusive. TC-007 and TC-020 validate this precisely.

### Decision 3: Explainability is a First-Class Feature
The requirements say testing the logging/explainability behavior is a "core part of this exercise." I created a dedicated Section 5 for explainability scenarios (TC-027, TC-028, TC-029) — not just checking the final price is correct, but verifying the log content is complete and accurate.

### Decision 4: Priority Rationale
- **P0** = Directly affects final price (wrong price = business impact)
- **P1** = Affects rule selection but caught by related P0 tests
- **P2** = Non-price output correctness (log completeness, error messages)
- **P3** = Cosmetic / formatting issues

---

*End of AI_Tools_Notes.md*
