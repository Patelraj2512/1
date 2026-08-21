# TestStrategy.md
## Discount / Pricing Rule Engine — QA Test Strategy

---

## 1. Overview & Purpose

The **Discount Rule Engine** is a core pricing service for an e-commerce/booking platform.
It computes the **final price** for a purchase by evaluating one or more discount rules
based on the customer's profile and purchase context.

**Why this needs rigorous testing:**
- Incorrect discounts = direct revenue loss or customer overcharging
- Rule conflicts must be resolved deterministically and *logged* for explainability
- Discount stacking vs. best-pick logic must be consistent
- Time-based rules (promotional windows) must not leak across date boundaries

---

## 2. Scope

### In Scope
| Area | Description |
|---|---|
| Rule evaluation | Each rule fires correctly for its configured attributes |
| Discount computation | Final price calculation (base price minus discount) |
| Rule conflict resolution | Stacking vs. highest-value vs. precedence-based winner |
| Logging / Explainability | Log captures which rule(s) fired and why |
| Negative / boundary inputs | Invalid, missing, zero, or out-of-range attributes |
| Date-range rules | Promotional windows activate/deactivate correctly |
| Quantity thresholds | Bulk discount triggers at exact threshold |

### Out of Scope
| Area | Reason |
|---|---|
| Payment gateway integration | Outside rule engine boundary |
| UI / front-end rendering | Not part of rule engine |
| Inventory management | Separate service |
| Tax computation | Separate concern; not mentioned in requirements |
| User authentication / login | Infrastructure layer |

---

## 3. Assumptions (Documented — Requirements Were Intentionally Vague)

| # | Assumption | Rationale |
|---|---|---|
| A1 | When two rules yield the **same discount value**, the rule with **higher configured precedence** wins. If precedence is also equal, the **most recently created rule** wins. | "Best price for the customer" is ambiguous when values are equal; precedence is the tiebreaker. |
| A2 | Rules **can stack** only if explicitly configured as stackable = true; otherwise the **highest-value rule wins** by default. | The system must decide — we assume opt-in stacking. |
| A3 | **Loyalty Tier** (Silver/Gold/Platinum) applies only to Loyalty Member customer type. New/Returning customers have no tier. | Tier without loyalty type is logically inconsistent. |
| A4 | **Date Range** is evaluated against the **server timestamp at time of purchase request**, in UTC. | Avoids timezone ambiguity. |
| A5 | A rule with inactive or expired status is **never evaluated**, even if attributes match. | Standard lifecycle behaviour. |
| A6 | Quantity threshold (e.g., "3+ items") means **quantity >= 3**. | Boundary: quantity = 3 should trigger; 2 should not. |
| A7 | Discounts are expressed as **percentage off base price** unless stated otherwise. | Most common e-commerce model. |
| A8 | The log/response must include: rule ID, rule name, discount applied, final price, and reason for selection (win/stack). | Explainability requirement. |

---

## 4. Test Levels

### 4.1 Unit Tests
**Responsibility:** Validate each rule in isolation — one rule, one attribute.

- Does a New Customer rule fire only for new customers?
- Does a Platinum tier rule fire only for Platinum members?
- Does a date-range rule fire only within its window?

**Tools:** xUnit / NUnit (C#) or pytest (Python)

---

### 4.2 Integration Tests
**Responsibility:** Validate the rule engine pipeline end-to-end — purchase context goes in, discounted price + log comes out.

- Multiple rules are loaded; the correct one(s) are selected.
- Conflict resolution logic works across combined inputs.
- Log output is correctly structured.

**Tools:** pytest with fixture-based context injection, or xUnit with dependency-injected rule repositories.

---

### 4.3 Functional / End-to-End Tests (API Level)
**Responsibility:** Validate the public API contract — what a real client would call.

- POST /calculate-price with a purchase context returns correct final_price, discounts_applied[], and log.
- Invalid inputs return appropriate error codes (400, 422).
- Edge cases return deterministic responses.

**Tools:** Postman / Newman — collections with pre-request scripts and test assertions.

---

### 4.4 Regression Tests
**Responsibility:** Ensure new rule configurations or code changes don't break existing behaviour.

- Automated suite run on every PR/merge.
- Covers all previously found defects.

---

## 5. Test Data Strategy

We construct purchase contexts as combinations of attributes to cover all rule dimensions:

| Dimension | Values Tested |
|---|---|
| Customer Type | New, Returning, Loyalty Member |
| Loyalty Tier | None, Silver, Gold, Platinum |
| Sales Channel | Web, Mobile App, Call Center, Agent |
| Quantity | 0, 1, 2, 3, 4, 10, 100 (boundary values) |
| Date | Before range, first day, last day, after range, mid-range |
| Rule Status | Active, Inactive, Expired |
| Discount Conflict | Single match, stack match, highest-value match, equal-value match |

**Combinations designed to trigger overlapping rules (conflict scenarios):**
- Loyalty Member + Platinum + Mobile App + Qty 5 + in seasonal window = 3+ rules fire simultaneously
- New Customer + Web + Qty 2 + out of date range = only customer-type rule fires
- Returning Customer + Web + Qty 3 + in promo window = channel + quantity + date rules overlap

---

## 6. Tools & Frameworks

| Tool | Purpose | Why |
|---|---|---|
| pytest (Python) | Unit + Integration tests | Concise, fixture-driven, great parametrize support for data-driven testing |
| xUnit / NUnit (C#) | Unit tests if codebase is .NET | Native, fast, CI-friendly |
| Postman / Newman | API / E2E testing | Readable, shareable collections; Newman enables CLI/CI execution |
| GitHub Actions / CI | Automated regression on PR | Catches regressions before merge |

---

## 7. Entry & Exit Criteria

### Entry Criteria (before testing begins)
- Rule Engine API is deployed to a test environment
- At least 3 rule configurations are seeded in the test database
- API contract / Swagger doc is available

### Exit Criteria (before sign-off)
- All P0 (Critical) and P1 (High) test cases pass
- 0 open Critical / High defects
- 100% traceability — every requirement maps to at least 1 test case
- All conflict-resolution and explainability scenarios have passing tests

---

## 8. Defect Triage & Reporting

| Severity | Definition | Example |
|---|---|---|
| P0 - Critical | Wrong final price returned | Gold tier gets New Customer discount |
| P1 - High | Conflict resolved incorrectly | Lower-value rule wins over higher-value |
| P2 - Medium | Log missing rule reason | Discount applied but reason field is null |
| P3 - Low | Minor formatting issues in log | Rule name has extra whitespace |

Defects reported in: JIRA / GitHub Issues with steps to reproduce, expected vs actual, purchase context JSON, and actual API response.

---

*End of TestStrategy.md*
