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
A rules engine recommends a margin move in percentage points based on observed
demand, and every recommended margin implies a price (`cost / (1 − margin)`).

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
| **Pricing** (`index.html`) | Region → Destination → Gateway accordion. Each row aggregates the departures in range — the Calendar view carries the date dimension. Filters, sorting, alerts, demand levels, margin, two adjustment inputs (ppt / £) usable at any level with propagation, and a read-only Sup column of committed margin. Accept, reject, reasons, notes and Ask work at every level. Bulk margin update, detail modals, Margin Copilot mock. |
| **Autopilot** (`autopilot.html`) | Same three-level accordion, where every row carries its own auto-accept margin band (±ppt) and peak-day switch, inherited gateway → destination → region. |
| **Alert rules** (`alerts.html`) | Same accordion again, carrying per-row alert thresholds (low pace, margin floor, conversion) with the same inheritance and per-row reset. |
| **Package rules** (`package-rules.html`) | Two-pane rule builder: scope a rule to a region, destination, gateway and departure type, add ALL/ANY conditions over demand, traffic, pace or margin, and set a margin action. Evaluated top to bottom, first match wins. |
| **Settings** (`settings.html`) | Global alert defaults that the per-row rules inherit from. |
| **Sign in** (`sso.html`) | Fake SSO screen; the drawer's log-out returns here. |

## Hierarchy

- **Region** — Western Mediterranean, Eastern Mediterranean, Canaries & Atlantic, Caribbean & Mexico
- **Destination** — 12 in total, each with a country (Faro/Algarve, Palma/Mallorca, Heraklion/Crete, Cancún, Montego Bay, …)
- **Gateway** — UK outbound departure point (London Gatwick, Manchester, Birmingham, Glasgow, …)

Parent rows are true roll-ups: volumes sum, rates are revenue- or
session-weighted, and margin is recomputed from summed revenue and summed cost
so it always reconciles with the rows beneath it.

## Demand levels

Observed-vs-expected demand maps to five levels (DL1–DL5), which drive the
recommended margin move:

| Level | Meaning | Margin move |
| --- | --- | --- |
| DL5 | Very high demand | +2.5 to +5.0 ppt |
| DL4 | High demand | +1.0 to +2.5 ppt |
| DL3 | Standard demand | hold |
| DL2 | Low demand | −1.0 to −2.5 ppt |
| DL1 | Very low demand | −2.5 to −5.0 ppt |

Most departures sit in DL3, so most rows carry no recommended change. Accept,
reject, reason codes, notes and Ask all work at region and destination level
too, so a whole region can be actioned in one click.

## Alerts

Low pace · Margin below floor · High demand · Low demand · Conversion drop ·
Out-of-line move.

Thresholds are set globally in Settings and can be overridden per region, per
destination or per gateway on the Alert rules page. The most specific override
wins; overridden values show in pink with a reset control.

## Running locally

No build step — it's static files:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000/>.

## Notes

- State (decisions, overrides, rules) is held in memory and `localStorage`, so
  "Commit" and rule changes persist within a browser but nothing leaves it.
- Demo "today" is 2026-07-15; the horizon runs 30 departure dates from 07-16.
- Some internal identifiers still carry names from the template this was
  adapted from (`PARKS`, `park`, `tickets`). That is deliberate — it keeps the
  shared render and aggregation code identical to its origin.
