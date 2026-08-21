# TraceabilityMatrix.md
## Discount / Pricing Rule Engine — Requirement Traceability Matrix

---

### Purpose of This Document

A **Traceability Matrix** maps every requirement (or scenario type) to the test cases that cover it.
This makes **coverage gaps instantly visible** — if a requirement row has no test case IDs, it's untested.

---

## Matrix

| REQ-ID | Requirement / Attribute | Scenario Type | Test Case IDs | Coverage Status |
|---|---|---|---|---|
| REQ-01 | CustomerType = New triggers New Customer discount | Positive | TC-001 | Covered |
| REQ-02 | CustomerType = Returning does NOT trigger New Customer discount | Negative | TC-002 | Covered |
| REQ-03 | CustomerType = Loyalty Member + Platinum tier triggers 20% discount | Positive | TC-003 | Covered |
| REQ-04 | CustomerType = Loyalty Member + Gold tier triggers 15% discount | Positive | TC-004 | Covered |
| REQ-05 | CustomerType = Loyalty Member + Silver tier triggers 10% discount | Positive | TC-005 | Covered |
| REQ-06 | Channel = Mobile App triggers channel discount | Positive | TC-006 | Covered |
| REQ-07 | Channel = Web produces no channel discount (no rule configured) | Negative | TC-002, TC-001 | Covered |
| REQ-08 | Quantity >= threshold triggers bulk discount (at-boundary) | Boundary | TC-007 | Covered |
| REQ-09 | Quantity < threshold does NOT trigger bulk discount (one below) | Boundary | TC-020 | Covered |
| REQ-10 | Quantity >> threshold still triggers bulk discount (well above) | Boundary | TC-021 | Covered |
| REQ-11 | Date within promotional window triggers seasonal discount | Positive | TC-008 | Covered |
| REQ-12 | Date = first day of promo window triggers discount (inclusive) | Boundary | TC-016 | Covered |
| REQ-13 | Date = last day of promo window triggers discount (inclusive) | Boundary | TC-017 | Covered |
| REQ-14 | Date = day before promo starts does NOT trigger discount | Boundary | TC-018 | Covered |
| REQ-15 | Date = day after promo ends does NOT trigger discount | Boundary | TC-019 | Covered |
| REQ-16 | INACTIVE rule never applies even on full attribute match | Negative | TC-011 | Covered |
| REQ-17 | EXPIRED rule never applies | Negative | TC-012 | Covered |
| REQ-18 | Missing required field (CustomerType) returns validation error | Negative | TC-009 | Covered |
| REQ-19 | Invalid field value returns error or graceful no-match | Negative | TC-010 | Covered |
| REQ-20 | Zero quantity rejected | Negative | TC-014 | Covered |
| REQ-21 | Negative quantity rejected | Negative | TC-015 | Covered |
| REQ-22 | LoyaltyTier on non-Loyalty-Member type is ignored | Negative | TC-013 | Covered |
| REQ-23 | When 2 rules match (no stacking): highest-value discount wins | Conflicting | TC-022 | Covered |
| REQ-24 | When a rule is stackable = true, it stacks on top of the winner | Conflicting | TC-023 | Covered |
| REQ-25 | Equal discount value: explicit precedence determines winner | Conflicting | TC-024 | Covered |
| REQ-26 | Three rules match simultaneously: correct winner selected | Conflicting | TC-025 | Covered |
| REQ-27 | Explicit precedence can override higher discount value | Conflicting | TC-026 | Covered |
| REQ-28 | Log includes rule ID, name, discount %, reason on single match | Explainability | TC-027 | Covered |
| REQ-29 | Log shows both applied and rejected rules on conflict | Explainability | TC-028 | Covered |
| REQ-30 | Log explicitly states "no rules matched" when no discount applies | Explainability | TC-029 | Covered |

---

## Coverage Summary

| Category | Requirements Count | Test Cases Count | Status |
|---|---|---|---|
| Positive Scenarios | 7 | 8 | Covered |
| Negative Scenarios | 7 | 7 | Covered |
| Boundary Scenarios | 6 | 7 | Covered |
| Conflicting-Rule Scenarios | 5 | 5 | Covered |
| Explainability Scenarios | 3 | 3 | Covered |
| **TOTAL** | **28** | **29** | **100% Covered** |

---

## Coverage Gaps (None Identified — but Watch For:)

| Risk Area | Notes |
|---|---|
| Call Center / Agent channel discounts | No specific rule configured in test data — if product team adds these rules, new test cases needed (TC-030+) |
| Stacking order when 3+ rules stack | Only tested with 2 stackable rules; more combinations may reveal ordering bugs |
| Negative base price (e.g., refund scenarios) | Not in scope currently — would need separate requirements |
| Concurrent requests with same rule | Load/race condition — not in scope for unit testing; would be a performance/load test concern |

---

## Assumption Traceability

The following assumptions were made where requirements were intentionally vague. Each assumption is traceable to test cases that validate that the assumption is correctly implemented.

| Assumption ID | Assumption | Test Cases |
|---|---|---|
| A1 | Equal discount values: precedence tiebreaker | TC-024 |
| A2 | Default = highest-value wins; stacking requires opt-in | TC-022, TC-023 |
| A3 | LoyaltyTier only valid for Loyalty Member CustomerType | TC-013 |
| A4 | Date evaluated against server UTC timestamp | TC-016, TC-017, TC-018, TC-019 |
| A5 | Inactive/expired rules are never evaluated | TC-011, TC-012 |
| A6 | Quantity threshold is >= (inclusive at boundary) | TC-007, TC-020 |
| A7 | Discounts are percentage-based | All TC-001 through TC-029 |
| A8 | Log must include ruleId, ruleName, discount, reason, finalPrice | TC-027, TC-028, TC-029 |

---

*End of TraceabilityMatrix.md*
