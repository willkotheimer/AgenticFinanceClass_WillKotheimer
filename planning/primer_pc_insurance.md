# P&C insurance primer for the capstone

Written for me. Anchor: everything below is an aggregation of the **case reserve** an adjuster
sets on a claim.

## 1. Reserves

- **Case reserve** — adjuster's estimate of a known claim's ultimate cost.
- **IBNR** — incurred but not reported: claims that happened but are not yet reported, plus
  expected drift on known claims. Set by actuaries. Pure estimate.
- **Unpaid losses and LAE** = case reserves + IBNR. Largest liability on a P&C balance sheet.
- **LAE** — loss adjustment expense: the cost of handling claims (ALAE per claim, ULAE overhead).

## 2. Ratios

- **Earned premium** — portion of written premium whose policy period has elapsed. Revenue.
- **Loss ratio** = (losses + LAE) ÷ earned premium.
- **Expense ratio** = underwriting expenses ÷ premium.
- **Combined ratio** = loss ratio + expense ratio. < 100 % = underwriting profit.
  > 100 % can still be profitable overall via investment income on the float.
- Insurtechs (ROOT, LMND, HIPO) ran well over 100 % for years; the story is the trend.

## 3. Reserve development — the honesty signal

Each year prior-year claims are re-estimated.
- Costing more than reserved → **adverse development**, charged to this year's income.
- Costing less → **favourable development**.
- Persistent adverse = under-reserving. Persistent favourable = conservative, or earnings smoothing.
- Disclosed in the **reconciliation of reserves** table: beginning → incurred (current year /
  **prior years** ← this is development) → paid → ending.

## 4. Loss development triangles

- Rows = accident year (when the loss occurred). Columns = age in months (12, 24, 36 …).
- Cell = cumulative incurred (or paid) losses for that accident year as of that age.
- Across a row: one year's claims being re-estimated as they mature. Down a column: years
  compared at the same age.
- **ASU 2015-09** (2016 onward): P&C 10-Ks must show ten years of incurred and paid triangles
  by major line, plus IBNR and claim counts, in the *Liability for Unpaid Losses* note.

## 5. Chain-ladder (deterministic — eval ground truth)

1. Age-to-age factor for each adjacent column pair = later ÷ earlier, averaged across the
   accident years that have both.
2. Multiply the latest diagonal forward through the factors to an **ultimate** per accident year.
3. Reserve = ultimate − paid to date.
- Tail factor for development beyond the last column: keep it a declared parameter.
- Agent's chain-ladder must match pytest's to the dollar.

## 6. Bootstrap (the C4 Monte Carlo tool)

Resample residuals of the chain-ladder fit → thousands of plausible triangles → chain-ladder
each → reserve distribution (P50 / P75 / P95). "Reserve variability." Recombines observed
history only; never extrapolates beyond the triangle. Reference: England & Verrall; Mack.

## 7. Reinsurance and catastrophe

- Insurer cedes risk to reinsurers. **Recoverables** = what reinsurers owe for incurred losses
  (an asset; concentration risk if one reinsurer dominates).
- **Quota share** — reinsurer takes a fixed % of every policy. Insurtechs used it heavily to
  grow without capital → gross vs. net loss ratios tell different stories. Ask for both.
- **Cat losses** — named events, disclosed separately. Florida writers (HCI, HRTG) are
  reinsurance-dependent; the June 1 cat program is the annual story.

## 8. Where it lives in the 10-K

| What | Where |
|---|---|
| Combined ratio, development commentary, cat losses | Item 7 MD&A |
| Reconciliation of reserves, triangles, IBNR | Item 8 note: *Liability for Unpaid Losses and LAE* |
| Reinsurance recoverables and concentration | Item 8 *Reinsurance* note |
| Cat, reserving, reinsurance-availability risks | Item 1A |
| Structured figures | XBRL, e.g. `LiabilityForClaimsAndClaimsAdjustmentExpense`, `PremiumsEarnedNet`, `IncurredClaimsPriorYears` — verify tags per filer |

## 9. Insurtech questions (the career-relevant three)

1. Is the loss ratio trending toward profitability?
2. Is reserve development credible? Young books have thin triangles; early favourable
   development is suspect.
3. How much risk is actually held vs. ceded?

## Reading

- CAS monograph, Friedland, *Estimating Unpaid Claims Using Basic Techniques* — free PDF from
  the Casualty Actuarial Society. Chapters 5–7 cover chain-ladder and related methods.
- Any roster company's 10-K, Item 8, the unpaid-losses note — read one end to end.
