# Insurance companies

P&C insurer roster for the insurance version of the Filings Research Desk (follow-up; see
`alternate_followup_ideas.md`, D-021 → D-022). All verified on EDGAR 2026-09-13: SIC 6331,
domestic 10-K filers, fiscal year end 31 Dec. Primer: `primer_pc_insurance.md`.

## Roster

| Ticker | Company | CIK | Line | 10-Ks | Latest 10-K | Why it is in |
|---|---|---|---|---|---|---|
| ROOT | Root, Inc. | 1788882 | insurtech auto | 6 | [2026-02-25](https://www.sec.gov/Archives/edgar/data/1788882/000178888226000014/root-20251231.htm) | my lane; 3 charter amendments (8-K 5.03) |
| LMND | Lemonade, Inc. | 1691421 | insurtech multi-line | 6 | [2026-02-25](https://www.sec.gov/Archives/edgar/data/1691421/000169142126000016/lmnd-20251231.htm) | growth-over-profit; young book, thin triangles |
| HIPO | Hippo Holdings Inc. | 1828105 | insurtech home | 6 | [2026-03-05](https://www.sec.gov/Archives/edgar/data/1828105/000182810526000008/hippo-20251231.htm) | 2 Nasdaq listing-deficiency notices (8-K 3.01) — the minefield |
| KINS | Kingstone Companies, Inc. | 33992 | NY homeowners | 18 | [2026-03-16](https://www.sec.gov/Archives/edgar/data/33992/000003399226000013/kins-20251231.htm) | tiny; 18 years of 10-Ks; 3 deficiency notices; **no XBRL development tag** — table only |
| HCI | Hci Group, Inc. | 1400810 | Florida homeowners | 18 | [2026-02-26](https://www.sec.gov/Archives/edgar/data/1400810/000119312526076743/hci-20251231.htm) | cat exposure; 6 charter amendments; longest tag history |
| HRTG | Heritage Insurance Holdings, Inc. | 1598665 | Florida homeowners | 12 | [2026-03-12](https://www.sec.gov/Archives/edgar/data/1598665/000119312526103715/hrtg-20251231.htm) | second view of the same cat risk; stale recoverables tag since 2018 |
| UFCS | United Fire Group Inc | 101199 | commercial P&C | 11 | [2026-02-26](https://www.sec.gov/Archives/edgar/data/101199/000010119926000015/ufcs-20251231.htm) | old-line clean baseline; 11 years of 10-Ks |
| KNSL | Kinsale Capital Group, Inc. | 1669162 | E&S specialty | 10 | [2026-02-20](https://www.sec.gov/Archives/edgar/data/1669162/000166916226000015/knsl-20251231.htm) | the well-run contrast; largest reserves on the roster |
| NODK | Ni Holdings, Inc. | 1681206 | regional, crop + farm | 10 | [2026-03-06](https://www.sec.gov/Archives/edgar/data/1681206/000117494726000305/nodk-20251231.htm) | small, different book |

## XBRL ground truth available (FY2025 unless noted)

Checked via `data.sec.gov/api/xbrl/companyfacts`. ✓ = tag present with a FY 10-K value.

| Ticker | Unpaid losses & LAE | Earned premium | Incurred losses (net) | Prior-year development (Sched. VI tag) | Recoverables — current tag |
|---|---|---|---|---|---|
| ROOT | ✓ $386M | ✓ $1,402M | — | ✓ | RRForUnpaidClaimsAndClaimsAdjustments, RRsOnPaidAndUnpaidLosses, RRsOnUnpaidLossesGross |
| LMND | ✓ $303M | ✓ $536M | ✓ $347M | ✓ | RRForUnpaidClaimsAndClaimsAdjustments, RRsOnPaidAndUnpaidLosses, RRsOnPaidLosses |
| HIPO | ✓ $420M | ✓ $380M | — | ✓ | RRForUnpaidClaimsAndClaimsAdjustments, RRsOnPaidAndUnpaidLosses, RRs, RRsOnPaidLosses |
| KINS | ✓ $141M | ✓ $187M | ✓ $84M | **none — table only** | RRForUnpaidClaimsAndClaimsAdjustments, RRsGross |
| HCI | ✓ $576M | ✓ $822M | ✓ $242M | ✓ | RRForUnpaidClaimsAndClaimsAdjustments, RRsOnPaidAndUnpaidLosses, RRsOnPaidLosses, RRsOnUnpaidLossesGross |
| HRTG | ✓ $579M | ✓ $794M | ✓ $313M | ✓ | RRForUnpaidClaimsAndClaimsAdjustments, RRsOnPaidAndUnpaidLosses |
| UFCS | ✓ $1,925M | ✓ $1,293M | ✓ $764M | ✓ | RRs, RRsOnPaidAndUnpaidLosses |
| KNSL | ✓ $2,891M | ✓ $1,576M | ✓ $891M | ✓ | RRForUnpaidClaimsAndClaimsAdjustments, RRsOnPaidAndUnpaidLosses, RRsOnPaidLossesGross |
| NODK | ✓ $138M | ✓ $271M | ✓ $201M | ✓ | RRForUnpaidClaimsAndClaimsAdjustments, RRs, RRsOnUnpaidLossesGross |

## Findings from the filings check

1. **Loss ratio is computable from XBRL for all nine** (incurred ÷ earned), except ROOT, which
   does not use `PolicyholderBenefitsAndClaimsIncurredNet` — find its incurred tag or read the
   income statement table.
2. **Prior-year reserve development** — the honesty signal — is tagged for 8 of 9 under the
   Schedule VI element `SupplementalInformationForPropertyCasualtyInsuranceUnderwriters
   PriorYearClaimsAndClaimsAdjustmentExpense`, not `IncurredClaimsPriorYears`. **Kingstone has
   no development tag**; the number exists only in the reconciliation-of-reserves table. That is
   a built-in C2 case: the figures agent must read the table because XBRL cannot be the oracle.
3. **Reinsurance recoverables have tag drift.** Filers switch among `ReinsuranceRecoverables`,
   `ReinsuranceRecoverablesOnPaidAndUnpaidLosses`, and
   `ReinsuranceRecoverableForUnpaidClaimsAndClaimsAdjustments` across years; the old tag goes
   stale (HRTG last used `ReinsuranceRecoverables` in 2018). The figures tool needs a
   normalisation map per concept — the data-side twin of model-label normalisation.
4. **Loss development triangles** are not in company-facts XBRL as a usable grid; they are in
   the unpaid-losses note (Item 8) and must be read from the filing. Chain-ladder ground truth
   is therefore built by hand from the table, once per company, and pinned as a fixture.
5. HCI and UFCS have tag history back to 2013 — the best candidates for ten-year questions.

## Question templates (insurance)

- Combined ratio for FY and its loss / expense split; change vs. prior year.
- Prior-year reserve development (favourable / adverse) for the last three years, cited to
  the reconciliation table — and, for KINS, table-only.
- Net ultimate loss for accident year N across successive triangles: did it drift?
- Reinsurance recoverables as a share of reserves; largest reinsurer concentration.
- Gross vs. net loss ratio (insurtechs: how much risk is ceded via quota share).
- Cat losses for the year and the named events.
- Listing-deficiency notices or charter amendments in 24 months (8-K Items 3.01 / 5.03).
- Top risk factors added or removed vs. prior 10-K (LLM-judged, citation-checked).

## Method

`https://www.sec.gov/files/company_tickers.json` → CIK; `data.sec.gov/submissions/CIK##########.json`
→ forms, 8-K items, latest 10-K accession; `data.sec.gov/api/xbrl/companyfacts/CIK##########.json`
→ tag availability and FY values. `User-Agent` header required; ≤ 10 requests/s.
