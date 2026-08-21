# UnitTestCases.md
## Discount / Pricing Rule Engine — Unit Test Cases

---

### How to Read This Document

Each test case contains:
- **ID** — Unique identifier (TC-XXX)
- **Description** — What is being tested and why
- **Preconditions / Rule Config** — What rules must exist in the system before the test
- **Input (Purchase Context)** — The JSON attributes sent to the rule engine
- **Expected Output** — discount(s) applied, final price, log/explanation
- **Priority** — P0 (Critical), P1 (High), P2 (Medium), P3 (Low)

**Base Price used throughout:** $100 (simplifies mental arithmetic)

---

## SECTION 1 — Positive Scenarios
> **Goal:** Verify each rule attribute correctly triggers its rule in isolation (one rule, one attribute at a time).

---

### TC-001 — New Customer Discount Applies

| Field | Value |
|---|---|
| **ID** | TC-001 |
| **Description** | A New Customer should receive the 10% New Customer discount. This verifies the CustomerType attribute works in isolation. |
| **Rule Config** | Rule R1: CustomerType = New → 10% discount, Status = Active |
| **Input** | CustomerType: "New", Channel: "Web", Quantity: 1, Date: 2026-06-15 (no promo active) |
| **Expected Discount** | 10% off → $10 discount |
| **Expected Final Price** | $90.00 |
| **Expected Log** | Rule R1 applied. Reason: CustomerType matched 'New'. Discount: 10%. |
| **Priority** | P0 |

**Why this matters:** The most basic customer-type rule. If this fails, no customer-type discount logic works.

---

### TC-002 — Returning Customer Gets No New-Customer Discount

| Field | Value |
|---|---|
| **ID** | TC-002 |
| **Description** | A Returning Customer must NOT receive the New Customer discount. Validates rule does not over-fire. |
| **Rule Config** | Rule R1: CustomerType = New → 10% discount |
| **Input** | CustomerType: "Returning", Channel: "Web", Quantity: 1, Date: 2026-06-15 |
| **Expected Discount** | None |
| **Expected Final Price** | $100.00 |
| **Expected Log** | No rules matched. Final price = base price. |
| **Priority** | P0 |

**Why this matters:** Rules that fire for the wrong customer type are a serious revenue risk.

---

### TC-003 — Platinum Loyalty Tier Discount

| Field | Value |
|---|---|
| **ID** | TC-003 |
| **Description** | A Loyalty Member with Platinum tier should receive a 20% loyalty discount. |
| **Rule Config** | Rule R2: CustomerType = Loyalty Member, LoyaltyTier = Platinum → 20% discount |
| **Input** | CustomerType: "Loyalty Member", LoyaltyTier: "Platinum", Channel: "Web", Quantity: 1, Date: 2026-06-15 |
| **Expected Discount** | 20% → $20 |
| **Expected Final Price** | $80.00 |
| **Expected Log** | Rule R2 applied. Reason: LoyaltyTier matched 'Platinum'. Discount: 20%. |
| **Priority** | P0 |

---

### TC-004 — Gold Loyalty Tier Discount

| Field | Value |
|---|---|
| **ID** | TC-004 |
| **Description** | Loyalty Member with Gold tier should receive a 15% discount (less than Platinum). |
| **Rule Config** | Rule R3: CustomerType = Loyalty Member, LoyaltyTier = Gold → 15% discount |
| **Input** | CustomerType: "Loyalty Member", LoyaltyTier: "Gold", Channel: "Web", Quantity: 1, Date: 2026-06-15 |
| **Expected Discount** | 15% → $15 |
| **Expected Final Price** | $85.00 |
| **Expected Log** | Rule R3 applied. Reason: LoyaltyTier matched 'Gold'. Discount: 15%. |
| **Priority** | P1 |

---

### TC-005 — Silver Loyalty Tier Discount

| Field | Value |
|---|---|
| **ID** | TC-005 |
| **Description** | Loyalty Member with Silver tier should receive a 10% discount. |
| **Rule Config** | Rule R4: CustomerType = Loyalty Member, LoyaltyTier = Silver → 10% discount |
| **Input** | CustomerType: "Loyalty Member", LoyaltyTier: "Silver", Channel: "Web", Quantity: 1, Date: 2026-06-15 |
| **Expected Discount** | 10% → $10 |
| **Expected Final Price** | $90.00 |
| **Expected Log** | Rule R4 applied. Reason: LoyaltyTier matched 'Silver'. Discount: 10%. |
| **Priority** | P1 |

---

### TC-006 — Mobile App Channel Discount

| Field | Value |
|---|---|
| **ID** | TC-006 |
| **Description** | Purchases made via Mobile App get a 5% channel discount. Validates channel-based rule. |
| **Rule Config** | Rule R5: Channel = Mobile App → 5% discount |
| **Input** | CustomerType: "Returning", Channel: "Mobile App", Quantity: 1, Date: 2026-06-15 |
| **Expected Discount** | 5% → $5 |
| **Expected Final Price** | $95.00 |
| **Expected Log** | Rule R5 applied. Reason: Channel matched 'Mobile App'. Discount: 5%. |
| **Priority** | P1 |

---

### TC-007 — Quantity Threshold Discount (Exactly at Threshold)

| Field | Value |
|---|---|
| **ID** | TC-007 |
| **Description** | Purchasing exactly 3 items (the threshold) should trigger the bulk discount. This is the critical boundary case. |
| **Rule Config** | Rule R6: Quantity >= 3 → 8% discount |
| **Input** | CustomerType: "Returning", Channel: "Web", Quantity: 3, Date: 2026-06-15 |
| **Expected Discount** | 8% → $8 |
| **Expected Final Price** | $92.00 |
| **Expected Log** | Rule R6 applied. Reason: Quantity (3) >= threshold (3). Discount: 8%. |
| **Priority** | P0 |

---

### TC-008 — Date Range Discount (Mid-Range Date)

| Field | Value |
|---|---|
| **ID** | TC-008 |
| **Description** | A purchase on a date inside the promotional window should receive the seasonal discount. |
| **Rule Config** | Rule R7: DateRange = 2026-06-01 to 2026-06-30 → 12% discount |
| **Input** | CustomerType: "Returning", Channel: "Web", Quantity: 1, Date: 2026-06-15 |
| **Expected Discount** | 12% → $12 |
| **Expected Final Price** | $88.00 |
| **Expected Log** | Rule R7 applied. Reason: PurchaseDate (2026-06-15) is within range [2026-06-01, 2026-06-30]. Discount: 12%. |
| **Priority** | P0 |

---

## SECTION 2 — Negative Scenarios
> **Goal:** Verify the engine rejects invalid inputs and does not apply discounts to unmatched contexts.

---

### TC-009 — Missing CustomerType Field

| Field | Value |
|---|---|
| **ID** | TC-009 |
| **Description** | A purchase context with no CustomerType field should not crash the engine — it should return a validation error. |
| **Rule Config** | Any active rule exists |
| **Input** | CustomerType: (missing), Channel: "Web", Quantity: 1, Date: 2026-06-15 |
| **Expected Output** | HTTP 400 / Error: "CustomerType is required" |
| **Expected Log** | Request rejected at validation. |
| **Priority** | P1 |

---

### TC-010 — Invalid CustomerType Value

| Field | Value |
|---|---|
| **ID** | TC-010 |
| **Description** | An unrecognised CustomerType (e.g., "VIP") should not match any rule and should be rejected. |
| **Input** | CustomerType: "VIP", Channel: "Web", Quantity: 1, Date: 2026-06-15 |
| **Expected Output** | HTTP 422 / Error: "Invalid CustomerType value: VIP" OR no rule matches, price = base |
| **Priority** | P2 |

---

### TC-011 — Inactive Rule Does NOT Apply

| Field | Value |
|---|---|
| **ID** | TC-011 |
| **Description** | Even if all attributes match, an inactive rule must never apply. This ensures rule lifecycle is respected. |
| **Rule Config** | Rule R1: CustomerType = New → 10% discount, Status = INACTIVE |
| **Input** | CustomerType: "New", Channel: "Web", Quantity: 1, Date: 2026-06-15 |
| **Expected Discount** | None |
| **Expected Final Price** | $100.00 |
| **Expected Log** | Rule R1 found but skipped: Status = INACTIVE. No matching active rules. |
| **Priority** | P0 |

---

### TC-012 — Expired Rule Does NOT Apply

| Field | Value |
|---|---|
| **ID** | TC-012 |
| **Description** | A rule whose date range has expired must not fire, even if other attributes match. |
| **Rule Config** | Rule R7: DateRange = 2026-01-01 to 2026-01-31 → 12% discount, Status = Active |
| **Input** | CustomerType: "Returning", Channel: "Web", Quantity: 1, Date: 2026-06-15 |
| **Expected Discount** | None |
| **Expected Final Price** | $100.00 |
| **Expected Log** | Rule R7 date range expired. No match. |
| **Priority** | P0 |

---

### TC-013 — Loyalty Tier on Non-Loyalty Customer (Invalid Combo)

| Field | Value |
|---|---|
| **ID** | TC-013 |
| **Description** | A New Customer with LoyaltyTier = Platinum is an invalid combination. The engine must ignore the tier and not apply loyalty discounts. |
| **Assumption Used** | A3: Loyalty Tier is only valid for Loyalty Member type. |
| **Input** | CustomerType: "New", LoyaltyTier: "Platinum", Channel: "Web", Quantity: 1, Date: 2026-06-15 |
| **Expected Output** | If R1 (New Customer) matches: 10% discount. Platinum tier is ignored. |
| **Expected Final Price** | $90.00 |
| **Expected Log** | Rule R1 applied. LoyaltyTier ignored: not applicable to CustomerType 'New'. |
| **Priority** | P1 |

---

### TC-014 — Zero Quantity Purchase

| Field | Value |
|---|---|
| **ID** | TC-014 |
| **Description** | A purchase with quantity 0 should be rejected as invalid (you cannot buy zero items). |
| **Input** | CustomerType: "New", Channel: "Web", Quantity: 0, Date: 2026-06-15 |
| **Expected Output** | HTTP 400 / Error: "Quantity must be >= 1" |
| **Priority** | P1 |

---

### TC-015 — Negative Quantity

| Field | Value |
|---|---|
| **ID** | TC-015 |
| **Description** | Negative quantities are logically impossible. Engine must reject them. |
| **Input** | CustomerType: "New", Channel: "Web", Quantity: -5, Date: 2026-06-15 |
| **Expected Output** | HTTP 400 / Error: "Quantity must be >= 1" |
| **Priority** | P1 |

---

## SECTION 3 — Boundary Scenarios
> **Goal:** Test exact edges — first/last day of promo, exact quantity threshold, zero-value edge cases.

---

### TC-016 — Date Range: First Day of Promotion

| Field | Value |
|---|---|
| **ID** | TC-016 |
| **Description** | A purchase on the FIRST day of the promotional window must trigger the promo discount. Off-by-one errors here would silently exclude the first day. |
| **Rule Config** | Rule R7: DateRange = 2026-06-01 to 2026-06-30 → 12% |
| **Input** | Date: 2026-06-01 |
| **Expected Discount** | 12% → $12 |
| **Expected Final Price** | $88.00 |
| **Priority** | P0 |

---

### TC-017 — Date Range: Last Day of Promotion

| Field | Value |
|---|---|
| **ID** | TC-017 |
| **Description** | A purchase on the LAST day of the promotional window must still trigger the discount. |
| **Rule Config** | Rule R7: DateRange = 2026-06-01 to 2026-06-30 → 12% |
| **Input** | Date: 2026-06-30 |
| **Expected Discount** | 12% → $12 |
| **Expected Final Price** | $88.00 |
| **Priority** | P0 |

---

### TC-018 — Date Range: Day BEFORE Promotion Starts

| Field | Value |
|---|---|
| **ID** | TC-018 |
| **Description** | A purchase one day before the promo starts must NOT trigger the discount. |
| **Rule Config** | Rule R7: DateRange = 2026-06-01 to 2026-06-30 → 12% |
| **Input** | Date: 2026-05-31 |
| **Expected Discount** | None |
| **Expected Final Price** | $100.00 |
| **Priority** | P0 |

---

### TC-019 — Date Range: Day AFTER Promotion Ends

| Field | Value |
|---|---|
| **ID** | TC-019 |
| **Description** | A purchase one day after the promo window must NOT trigger the discount. |
| **Input** | Date: 2026-07-01 |
| **Expected Discount** | None |
| **Expected Final Price** | $100.00 |
| **Priority** | P0 |

---

### TC-020 — Quantity: One Below Threshold (Should NOT Trigger)

| Field | Value |
|---|---|
| **ID** | TC-020 |
| **Description** | Quantity = 2 when threshold is 3 must NOT trigger the bulk discount. |
| **Rule Config** | Rule R6: Quantity >= 3 → 8% |
| **Input** | Quantity: 2 |
| **Expected Discount** | None |
| **Expected Final Price** | $100.00 |
| **Priority** | P0 |

---

### TC-021 — Quantity: Well Above Threshold

| Field | Value |
|---|---|
| **ID** | TC-021 |
| **Description** | Quantity = 100 should still correctly trigger the discount (no upper-bound bug). |
| **Rule Config** | Rule R6: Quantity >= 3 → 8% |
| **Input** | Quantity: 100 |
| **Expected Discount** | 8% → $8 |
| **Expected Final Price** | $92.00 |
| **Priority** | P2 |

---

## SECTION 4 — Conflicting Rule Scenarios
> **Goal:** Verify the engine correctly resolves situations where 2+ rules match the same purchase context.

---

### TC-022 — Two Rules Match: Highest-Value Rule Wins (Default)

| Field | Value |
|---|---|
| **ID** | TC-022 |
| **Description** | Loyalty Member (Platinum, 20%) + Mobile App channel (5%) both match. By default (no stacking), the higher-value rule (20%) should win. |
| **Rule Config** | R2: Platinum → 20%, R5: Mobile App → 5%, both non-stackable |
| **Input** | CustomerType: "Loyalty Member", LoyaltyTier: "Platinum", Channel: "Mobile App", Qty: 1 |
| **Expected Discount** | 20% (Platinum wins) |
| **Expected Final Price** | $80.00 |
| **Expected Log** | Rules R2 and R5 matched. Conflict resolved: R2 wins (highest discount value = 20% > 5%). R5 not applied. |
| **Priority** | P0 |

---

### TC-023 — Two Rules Stack (Stackable Config)

| Field | Value |
|---|---|
| **ID** | TC-023 |
| **Description** | When R5 (Mobile App, 5%) is configured as stackable = true, it should stack on top of R2 (Platinum, 20%). |
| **Rule Config** | R2: Platinum → 20% (non-stackable), R5: Mobile App → 5% (stackable = true) |
| **Input** | CustomerType: "Loyalty Member", LoyaltyTier: "Platinum", Channel: "Mobile App", Qty: 1 |
| **Expected Discount** | 20% + 5% = 25% → $25 |
| **Expected Final Price** | $75.00 |
| **Expected Log** | R2 applied (highest-value winner, 20%). R5 stacked (stackable = true, 5%). Total discount: 25%. |
| **Priority** | P0 |

---

### TC-024 — Equal Discount Value: Precedence Tiebreaker

| Field | Value |
|---|---|
| **ID** | TC-024 |
| **Description** | R1 (New Customer, 10%) and R4 (Silver, 10%) both yield 10% discount. R1 has higher precedence (precedence = 1 vs precedence = 5). R1 must win. |
| **Assumption Used** | A1: Explicit precedence is the tiebreaker for equal discount values. |
| **Rule Config** | R1: New Customer → 10%, Precedence = 1 (highest). R4: Silver → 10%, Precedence = 5. |
| **Input** | CustomerType: "Loyalty Member", LoyaltyTier: "Silver", ... (hypothetical multi-match) |
| **Expected Log** | Rules R1 and R4 matched with equal discount (10%). Conflict resolved by precedence: R1 (precedence=1) wins. |
| **Priority** | P1 |

---

### TC-025 — Three Rules Match: Channel + Loyalty + Seasonal

| Field | Value |
|---|---|
| **ID** | TC-025 |
| **Description** | Maximum overlap scenario: Platinum (20%) + Mobile App (5%) + June promo (12%) all match. Only highest-value wins unless stacking is configured. |
| **Rule Config** | R2: Platinum → 20%, R5: Mobile App → 5%, R7: June promo → 12%. None stackable. |
| **Input** | CustomerType: "Loyalty Member", LoyaltyTier: "Platinum", Channel: "Mobile App", Qty: 1, Date: 2026-06-15 |
| **Expected Discount** | 20% (Platinum wins) |
| **Expected Final Price** | $80.00 |
| **Expected Log** | Rules R2, R5, R7 all matched. Conflict resolved: R2 wins (highest value 20%). R5 (5%) and R7 (12%) not applied. |
| **Priority** | P0 |

---

### TC-026 — Explicit Precedence Overrides Higher Discount Value

| Field | Value |
|---|---|
| **ID** | TC-026 |
| **Description** | R8 has a lower discount (8%) but is explicitly set as Precedence = 1 (highest priority). R2 has 20% but Precedence = 10. R8 must win per the documented precedence order. |
| **Assumption Used** | A1 second part: when precedence is explicitly configured, it can override discount value ranking. |
| **Rule Config** | R8: Agent Channel → 8%, Precedence = 1. R2: Platinum → 20%, Precedence = 10. |
| **Input** | CustomerType: "Loyalty Member", LoyaltyTier: "Platinum", Channel: "Agent", Qty: 1 |
| **Expected Discount** | 8% (R8 wins by precedence) |
| **Expected Log** | Rules R8 and R2 matched. R8 selected: explicit precedence (1) overrides higher discount value (20%). |
| **Priority** | P1 |

---

## SECTION 5 — Explainability Scenarios
> **Goal:** Verify that the log/response contains the correct rule(s) and reasoning — not just the right final price.

---

### TC-027 — Log Contains Rule ID and Reason for Single Match

| Field | Value |
|---|---|
| **ID** | TC-027 |
| **Description** | When one rule matches, the response log must include the rule ID, rule name, discount %, and reason. |
| **Input** | New Customer, Web, Qty 1, no promo date |
| **Expected Log Fields** | ruleId: "R1", ruleName: "New Customer Discount", discountApplied: 10, reason: "CustomerType = New", finalPrice: 90 |
| **Priority** | P0 |

---

### TC-028 — Log Contains Rejected Rules on Conflict

| Field | Value |
|---|---|
| **ID** | TC-028 |
| **Description** | When conflict resolution occurs, the log must show ALL matched rules and indicate which was selected and which were rejected, with reasons. |
| **Input** | Platinum Loyalty Member + Mobile App (two rules match, highest-value wins) |
| **Expected Log Fields** | appliedRule: {R2, 20%, reason: highest-value winner}, rejectedRules: [{R5, 5%, reason: lower value}] |
| **Priority** | P0 |

---

### TC-029 — No Rules Matched: Log Confirms Zero Discount

| Field | Value |
|---|---|
| **ID** | TC-029 |
| **Description** | When no rule matches, the log must explicitly state no rules matched (not just return base price silently). |
| **Input** | Returning Customer, Call Center, Qty 1, no promo |
| **Expected Log** | rulesEvaluated: 5, rulesMatched: 0, discountApplied: 0, reason: "No active rules matched purchase context", finalPrice: 100 |
| **Priority** | P1 |

---

*End of UnitTestCases.md*
