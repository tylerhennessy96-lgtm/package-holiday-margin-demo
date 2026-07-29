# Dynamica SmartRates — package-holiday margin demo

A clickable UI demo of margin management for a package-holiday operator
(flights + hotels sold as packages). Everything runs in the browser on
**synthetic data generated client-side by a seeded RNG** — deterministic across
reloads, no backend, no build step, no dependencies.

**[▶ Open the demo](https://tylerhennessy96-lgtm.github.io/package-holiday-margin-demo/)**

> All numbers, destinations and prices are illustrative. No real commercial data
> is used or implied.

## What it shows

Margin, not price, is the lever. Each row carries a package cost per person
(contracted flight seat + hotel) against a selling price, giving a margin %.

The **target margin is built additively**: a 12% base plus one adjustment from
each of six factor groups — competitive position, demand and conversion, basket
value, booking channel, departure window, and supplier cost advantage — then
clamped by guardrails (6–18% margin, ±6 ppt movement, £75 minimum contribution
per booking). The detail modal shows the whole calculation line by line.

The adjustments are deliberately flat and additive to start with. The intent is
that Dynamica then learns whether each step should be larger, smaller or more
granular from observed conversion and contribution, exploring within ±1.5 ppt
of the calculated target.

Demand comes from **observed web traffic** — sessions and conversion. The
operator does not own the flights, so there is no seat capacity, load factor or
booking curve anywhere in the model.

**Adjustments** sit on every row at every level: margin points (ppt) and a
price move (£ per person). Edit one on a region or destination and it
propagates to every product beneath it. A parent shows a value only when its
children agree — otherwise it reads "mixed" rather than inventing an average.

**Sup** is read-only. Committing folds the pending adjustment into it and
clears the inputs, so it accumulates a running total of the margin added on top
of the engine's baseline.

## Pages

| Page | What it does |
| --- | --- |
| **Pricing** (`index.html`) | Region → Destination → Lead time accordion. Each row aggregates the departures in range — the Calendar view carries the date dimension. Filters (including revenue manager, so an RM sees only their own book), sorting, alerts, demand levels, margin, two adjustment inputs (ppt / £) usable at any level with propagation, and a read-only Sup column of committed margin. Accept, reject, reasons, notes and Ask work at every level; accepting moves the live margin to the recommendation. Bulk margin update, detail modals, Margin Copilot mock. |
| **Autopilot** (`autopilot.html`) | Same three-level accordion, where every row carries its own auto-accept margin band (±ppt) and peak-day switch, inherited gateway → destination → region. |
| **Alert rules** (`alerts.html`) | Same accordion again, carrying per-row alert thresholds (low pace, margin floor, conversion) with the same inheritance and per-row reset. |
| **Margin rules** (`package-rules.html`) | The additive matrix itself: every factor band and every guardrail is editable, with a live worked example that recalculates as you change them. Feeds the Pricing engine. |
| **Settings** (`settings.html`) | Global alert defaults that the per-row rules inherit from. |
| **Sign in** (`sso.html`) | Fake SSO screen; the drawer's log-out returns here. |

## Hierarchy

- **Region** — Western Mediterranean, Eastern Mediterranean, Canaries & Atlantic, Caribbean & Mexico
- **Destination** — 12 in total, each with a country (Faro/Algarve, Palma/Mallorca, Heraklion/Crete, Cancún, Montego Bay, …)
- **Lead time** — each destination splits into ≤7d · 8–30d · 31–90d · 91–180d · 180d+, so a destination is never viewed across all time at once. It has its own column and filter.
- **Gateway** — UK outbound departure points still drive cost and price underneath, but they are aggregated into the destination rather than shown as a level.

Parent rows are true roll-ups: volumes sum, rates are revenue- or
session-weighted, and margin is recomputed from summed revenue and summed cost
so it always reconciles with the rows beneath it.

## The margin matrix

| Factor | Range |
| --- | --- |
| Competitive position — price vs competitor benchmark | +2.0 → −2.0 ppt |
| Demand and conversion — vs conversion target | +2.0 → −2.0 ppt |
| Basket value — total package value per booking | +1.5 → −1.0 ppt |
| Booking channel — blended across the acquisition mix | −0.5 → +1.5 ppt |
| Departure window — days until departure | +1.5 → −1.0 ppt |

The departure-window bands are the same definition the table splits rows by, so
a row maps to exactly one band and the factor is exact rather than blended — a
destination's target steps down through the five bands purely on lead time.
| Supplier cost advantage — cost vs benchmark | +2.0 → −1.0 ppt |

Worked example from the rule sheet: 12.0% base, +1.0 (4% cheaper than
competitors), +2.0 (high demand, strong conversion), −0.5 (£3,500 basket),
+0.5 (paid search), +1.0 (departing in 20 days), +1.0 (supplier cost 5% better)
= **17.0% target margin**.

Observed demand still maps to five levels (DL1–DL5) and feeds the demand
factor. Accept,
reject, reason codes, notes and Ask all work at region and destination level
too, so a whole region can be actioned in one click.

## Alerts

Low pace · Margin below floor · High demand · Low demand · Conversion drop ·
Out-of-line move.

Thresholds are set globally in Settings and can be overridden per region, per
destination or per gateway on the Alert rules page. The most specific override
wins; overridden values show in pink with a reset control.

An alert only rolls up to a row once it affects a material share of the
departures beneath it, and that share is set per alert type — a thin margin on
a single departure is ordinary, an out-of-line move is not. Parent rows then
summarise the rows beneath them ("2 of 5 lead-time bands"), so the screen
flags what is worth looking at instead of lighting up everywhere.

## Running locally

No build step — it's static files:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000/>.

## Notes

- State (decisions, overrides, rules) is held in memory and `localStorage`, so
  "Commit" and rule changes persist within a browser but nothing leaves it.
- Demo "today" is 2026-07-15; the horizon runs 365 departure dates from 07-16,
  so all five lead-time bands populate.
- Some internal identifiers still carry names from the template this was
  adapted from (`PARKS`, `park`, `tickets`). That is deliberate — it keeps the
  shared render and aggregation code identical to its origin.
