<p align="center"><img src="assets/banner.svg" alt="Inventory &amp; Reorder Planner" width="100%"></p>

# Inventory & Reorder Planner: Client Case Study

> Know what to order, when and how much, and see why.

> **Status of this document.** This is a reference case study: it sets out the business problem, the solution and the value model. The results table is intentionally blank and is filled in only from measured data (a pilot or a real engagement). No outcome figures are claimed here. The planner behind it exists as a working local prototype on synthetic data: it has never held a real shop's stock, has no authentication, and **cannot place an order, send an email or contact a supplier**. Experiments on simulated demand are in the [research report](research-report.md) and are not client outcomes. **The planner turns the sales and supplier terms that people record into forecasts and suggested quantities. They are statistical estimates, not guarantees, and every suggested order is for a person to review.**

## Snapshot

|  |  |
|---|---|
| **Who it is for** | Owners and buyers of small retailers and workshops (roughly 1 to 30 people) that carry from a few dozen to a few thousand stocked items from several suppliers and reorder by eye, by spreadsheet or by habit. |
| **Solution** | A stock ledger that knows what is on hand, on order and reserved; a forecast of demand per item; a safety stock and reorder point that reflect how long and how variable each supplier's delivery is; and a short, explained list of what to order, grouped into purchase-order drafts per supplier. |
| **Track** | Business |
| **Delivered by** | The Diamond Empire (https://thedempire.shop/) |

## The challenge

Reordering is done from memory and from the shelf. Fast sellers run out because the supplier is slower than anyone remembers; slow sellers pile up because "we might need it"; a bulk deal quietly buys a year of stock. Minimum order quantities and pack sizes make the right quantity awkward to work out, so people round up and tie cash in the stockroom. Questions as simple as "what should I order from this supplier today?", "how much money is sitting in things that have not sold for six months?" and "how many sales did we lose to empty shelves?" take an afternoon with a spreadsheet, and the answers are out of date by Friday.

## The solution

A single-tenant planner. Every sale, receipt, return, write-off and stock adjustment is a line in an append-only ledger, so on-hand, on-order and reserved quantities can always be rebuilt and checked. For each item the planner forecasts demand (a method is chosen from the shape of the sales history, or fixed by the user), estimates how uncertain that forecast is, combines it with the supplier's lead time and its variability, and sets a reorder point for the service level the owner picked. When stock plus what is already on order falls to that point, the item appears in a short list with the arithmetic behind it. Suggestions become purchase-order drafts grouped by supplier, rounded to pack sizes and minimum order quantities. A person approves, skips or adjusts each line, and every decision is recorded with who made it and why.

What the client gets in practice:

- Items, locations, suppliers and per-supplier terms (lead time with its variability, pack size, minimum order quantity, unit cost), and a stock ledger with on-hand, on-order and allocated balances that can be replayed from the ledger to prove they are right
- Demand forecasting with four methods (moving average, exponential smoothing, seasonal-naive, Croston for intermittent items), safety stock and reorder point calculators, economic order quantity with pack and minimum rounding, ABC and XYZ classification, and flags for dead stock and overstock
- Reorder suggestions that show their reasons, purchase-order drafts per supplier, approve / skip / adjust with an audit trail, and a local outbox that holds draft emails (**nothing is ever sent**)
- A local web UI (dashboard, item page with a demand sparkline, supplier pages, PO drafts, reports) and a JSON API, strict CSV import with per-row errors, and CSV export
- Reports: stock value, turnover, estimated holding cost and lost sales, and forecast accuracy by method on the client's own history
- A seeded synthetic demand generator and an inventory simulator that runs the real planning code, so a policy change can be tried on simulated demand before it touches real orders

## How it creates value

| Value lever | How the solution delivers it | How we measure it |
|---|---|---|
| Fewer stockouts | Reorder points that reflect lead time, lead-time variability and demand variability for a chosen service level | Fill rate, number of stockout events and estimated lost sales on the items in scope |
| Less cash tied up | Order quantities from cost, not habit; dead stock and overstock flagged; reorder points that do not carry more safety stock than needed | Average inventory value, turnover, value of dead stock |
| Less buying time | One ranked list instead of a walk round the shelves; drafts already grouped by supplier and rounded to packs | Time from "what do we need?" to a reviewed draft order |
| Confidence in the numbers | Every suggestion shows demand, lead time, safety stock, position and quantity; balances can be replayed from the ledger | Share of suggestions accepted unchanged; inventory-count adjustments over time |
| Supplier terms in one place | Supplier pages list the lead time, its variability, pack size, minimum order quantity and unit cost on record for every item they supply | Share of items with complete supplier terms; terms corrected after the first review |

## How an engagement runs

| Phase | What happens |
|---|---|
| Model | Capture items, locations, suppliers and terms; load stock and sales history from the client's spreadsheets through the CSV import, which reports every problem row |
| Check | Compare the planner's forecasts and reorder points with the buyer's own judgement on the fast and slow movers, correct wrong terms (lead times are the usual culprit) and set service levels by item importance. The planner does not know the business: this review is the control |
| Connect | Link the client's point of sale, accounting and supplier email or ordering channel, and add sign-in. Engagement work, not built |
| Pilot | One supplier group or product category for one or two buying cycles, with a before/after comparison of the measures above |
| Expand | Remaining categories, more locations, tuned policies and reports |

## What the client receives

- A configured instance loaded with the client's items, locations, suppliers, terms and history
- Training for the buyers and for the person who maintains the data
- A baseline report (stock value, turnover, dead stock, forecast accuracy on the client's history) taken at go-live
- Integration of the client's point of sale, email and sign-in (engagement work: the prototype has none of these)

## Results

Baseline measurements are taken **before** the solution goes live, and the same measures are taken after, so every change in the table is attributable and honest.

| Measure | How measured | Baseline | After | Change |
|---|---|---|---|---|
| Fill rate on in-scope items | Units sold from stock over units demanded, over one buying season | — | — | — |
| Stockout events | Number of occasions an in-scope item had nothing on hand while customers asked for it | — | — | — |
| Average inventory value | Mean daily on-hand value at cost | — | — | — |
| Inventory turnover | Cost of goods sold over average inventory value, annualised | — | — | — |
| Value of dead stock | Stock at cost with no sale for the dead-stock threshold | — | — | — |
| Buying time | Median minutes from starting a buying round to a reviewed draft order | — | — | — |

> **To fill in:** replace each — with measured values and add the measurement window and sample size beneath the table.

## How quality and risk are managed

- Automated tests cover the ledger and its replay, parsing and validation, the forecasting methods, the safety-stock, reorder-point and order-quantity calculators, the purchase-order workflow, the CSV importer and exporter, and the web UI and API; counts are in the README
- The calculators are checked against independent references (a brute-force Monte Carlo of the lead-time-demand model, exact enumeration for Poisson demand, brute-force search for order quantities), and the check suite is proven able to catch seventeen kinds of deliberate bug (experiment E3 in the research report)
- Forecast methods are compared on generated demand of five patterns with paired seeds and rolling-origin evaluation, and the report says which method wins where and where nothing is distinguishable (experiment E1)
- Reorder policies are compared in a simulator that calls the product's own planning code, and the report says whether the fill rate achieved matches the target (experiment E2)
- Every approval, skip and adjustment is in an audit trail the database refuses to edit; the ledger refuses edits and deletes too
- Everything runs locally on `127.0.0.1`; the demo contains only invented items and suppliers, and the outbox never sends anything
- Known limits are stated plainly in the README and the research report: no authentication, no real email or point-of-sale link, demand taken from recorded sales (stockouts hide demand), a normal approximation that is weak for very low-volume items, and simulated rather than real behaviour in the policy experiment

## Is this a fit?

Best suited to a small business that wants a clear, inspectable planner it can own and adapt, that records sales and stock movements (or can import them), and that is willing to have a point-of-sale link, supplier email and sign-in added in an engagement and to review the first suggestions with the people who know the stock. It is **not** a warehouse-management or ERP system: it does not manage bills of materials, lots and expiry dates, barcode scanning, multi-warehouse transfers, promotions or supplier price breaks, and it does not place orders.

## Talk to us

Want this adapted to your business? Get in touch via **[thedempire.shop](https://thedempire.shop/)**.

---

[README](../README.md) · [Client case study](case-study.md) · [Research report](research-report.md)
