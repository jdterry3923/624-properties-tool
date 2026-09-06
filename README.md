# Rental Underwriter

A single-file underwriting tool for residential rental acquisitions, built for seller-financed deals where price and terms are both negotiable.

Open `index.html` in any browser, or use the hosted page. No install, no build step, no dependencies. All calculations run client-side — nothing is transmitted or stored.

---

## Do not commit real deal data

The tool ships with generic placeholder numbers. Keep it that way.

Git history is permanent. A file committed with real prices, rents, addresses, or a seller's name stays in the history even after a later commit removes it — and on a public repo, that history is public. Stripping the data out later does not undo it.

Enter real numbers in the browser at runtime. They never touch the file.

---

## What it does

Five modules, all driven off one set of assumptions:

**Doors** — Per-property underwriting with a full income statement: gross rent through vacancy, operating expenses, NOI, capex reserve, debt service, and cash flow. Plus the cash required to acquire, broken into down payment, closing, make-ready, and held reserves.

**What would make it work** — When a deal fails, each lever is solved independently to the exact value that clears the coverage gate: purchase price, rent, note rate, amortization, down payment, operating load. One at a time, everything else constant.

**Balloon** — Tests whether the deal can refinance when a seller-carried note comes due. Runs at the note rate plus a stress bump, against both an LTV ceiling and the coverage gate. Reports the cash shortfall at refi, plus the rate or appraised value that would be required to clear it. Set the balloon to 0 for a fully amortizing note and the tab reports no refi risk.

**Capacity** — Works backward from available cash. Given a typical door profile, returns how many doors the cash supports, total debt at that scale, and the maximum price and minimum rent that clear the gates.

**Partners** — Preferred return with accrual when cash flow can't cover it, annual residual split, and a sale waterfall: return of capital, pref catch-up, then residual split. Reports the partner's total distributions and equity multiple, and flags capital calls and loss allocation.

---

## Definitions

Two DSCR figures are shown because they differ meaningfully.

**Lender DSCR** is NOI ÷ debt service. This is what a bank computes. Capex reserves are not deducted.

**Reserve-adjusted DSCR** is (NOI − capex reserve) ÷ debt service. This is the gate. It runs roughly 0.20–0.30 stricter than the lender figure, because the capital set aside for roofs and HVAC is not actually available to service debt.

**Cash-on-cash** is annual cash flow ÷ total cash invested, where total cash invested includes held reserves. Reserves are counted because that capital is committed and can't be deployed elsewhere.

---

## Default gates and policy

| Setting | Default | Why |
|---|---|---|
| Reserve-adjusted DSCR | 1.25x | Strict by design. Approximately equal to a 1.45–1.55x lender DSCR. |
| Cash-on-cash | 8% | Year one, after reserves. |
| Reserves held | 3 months PITI + $3,000 per door | Leaner than a 6-month policy; the trade-off is less runway on a long vacancy. |
| Expense growth | 3%/yr vs. 2% rent growth | Deliberately unfavorable. Fixed costs are assumed to outrun rents. |
| Appreciation | 1.5%/yr | No thesis should depend on the market rising. |
| Refi stress | Note rate + 2.00 pts | Applied in the balloon test. |

All are editable in the assumptions rail. Nothing is hardcoded.

**Note on the DSCR gate:** at 20% down and a 7% note, a house at a 1.0% rent-to-price ratio will not clear 1.25x. Clearing it requires roughly 1.35–1.45%. That is not a bug — it means the deal has to come from price or terms, not from the market. If the gate produces NO-GO on everything, the question is whether the standard is right for the situation, not whether the tool is working.

---

## How the projections grow

Multi-year figures (the balloon test and the partner waterfall) split operating expenses in two:

- **Fixed costs** — taxes, insurance, HOA — inflate at the expense growth rate.
- **Variable costs** — management and maintenance — are a percentage of effective gross income, so they track EGI rather than inflating separately.

Growing the whole expense line at the inflation rate would move the variable portion twice, once through EGI and again through inflation. The split avoids that.

---

## Partner structure notes

**Preferred return accrues simple, not compounding.** Unpaid pref adds to the balance owed but does not itself earn the pref rate. Most institutional pref compounds. If your partner assumes compounding and the model assumes simple, you have a disagreement to settle before the money comes in.

**Loss allocation is a setting, not an assumption.** If the sale nets a loss, the model can allocate it pro-rata by capital or have the sponsor absorb all of it. Pro-rata is the default. Pick deliberately — the two produce very different outcomes for the partner and the difference is invisible until things go wrong.

**Negative cash flow means capital calls.** When projected cash flow is negative, the Partners tab says so and names the worst year's shortfall. The model does not simulate call mechanics; decide per-call and lifetime caps before signing.

---

## Verify before trusting output

Three inputs move the answer more than everything else combined:

1. **Property taxes** — pull actual bills from the county appraisal district. No homestead exemption applies to a rental, and the tool's default percentage is an estimate only.
2. **Insurance** — get real quotes with the wind/hail deductible in writing. Hail exposure makes this line unpredictable.
3. **Rent** — use current lease amounts on occupied properties. Estimates from listing sites are not comps.

On a mid-priced house, taxes and insurance together can run 35–40% of effective gross income before a dollar of management, maintenance, or debt service. Getting them wrong on the optimistic side turns a 1.25x deal into a 1.05x deal.

---

## Deploying

Static hosting, no configuration needed. The page is a single `index.html` with no external dependencies.

This repo is public, which means the URL is public. The file includes a `noindex` meta tag so search engines skip it, but that is not access control. Anyone with the link can open it. Since the tool ships with placeholder numbers and computes entirely in the browser, that exposes the model — not any deal.

---

## Not yet built

- **Excel export** — the current Download button produces CSV. A formatted workbook for lenders and partners is the next piece.
- **Flip-vs-rent toggle** — same property, both exits, side by side. Should share inputs with the existing flip model so the two agree.
- **Multifamily** — different valuation logic (cap rate rather than comps), unit mix, and commercial expense ratios. Deliberately deferred; jamming it into the SFR model would compromise both.
- **Deal-level sensitivity grid** — price × rent, the way the flip model does ARV × overrun.
- **Saved scenarios** — the tool holds no state between page loads. Every session starts from defaults.

---

## Not advice

This is a modeling tool. Entity structure, the note itself, installment-sale treatment on the seller's side, and anything binding belong with a CPA and a real estate attorney licensed in the relevant state.
