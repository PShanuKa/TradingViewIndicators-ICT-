# TradingView Indicators - ICT

ICT (Inner Circle Trader) concepts for TradingView.

| File | Version | Status |
|---|---|---|
| [`ICT_v5.pine`](ICT_v5.pine) | **v5.2** (Pine v6) | Current - display + entry engine |
| [`ICT_v4.pine`](ICT_v4.pine) | v4.0 | Parked - drawings misaligned, unresolved |
| [`ICT_v3.1.pine`](ICT_v3.1.pine) | v3.1 | Reference only - statistics engine has known defects |
| [`ICT_v3.pine`](ICT_v3.pine) | v3.0 | Legacy |

## v5 - one verified layer at a time

v5 is a rebuild. Nothing is added until the layer under it is confirmed correct
on a live chart.

| Layer | What it does |
|---|---|
| 1. Order Blocks | Last opposing candle before a displacement break; optional breaker flip |
| 2. Fair Value Gaps | 3-candle imbalance, auto-fill |
| 3. Market Structure | Swing points, BOS (continuation), MSS (reversal) |
| 4. Liquidity & Hunts | EQH/EQL pools - **SWEEP** (wick + close back inside) told apart from **BREAK** |
| 5. Premium / Discount | Dealing range, equilibrium, OTE band |
| 6. Killzones | New York time sessions, DST automatic |
| 7. Entry Engine | The ICT sequence turned into signals with entry / SL / TP and alerts |

Every layer has its own on/off switch. If the chart ever looks wrong, switch
layers off one at a time - the culprit shows up in seconds.

### Entry model

    liquidity hunt  ->  displacement leaves an OB / FVG  ->  price returns to it
    ...inside a killzone, on the correct side of the dealing range,
       with structure already shifted in that direction.

Each requirement is a checkbox. **What you tick is what must be true** - untick
to loosen, tick to tighten. The score on the signal label is informational and
counts all six context conditions regardless of what is required.

Targets aim at the nearest *untouched* liquidity pool. If the resulting R:R is
below the minimum the setup is **skipped**, not stretched to make the number
look better.

### Not built yet

No backtest tracker and no win-rate panel. That is the next step, deliberately -
the v3.1 audit showed a measurement engine can be more wrong than the strategy
it measures.

> TradingView allows 500 boxes / 500 lines / 500 labels per script. "Unlimited"
> settings mean "up to that ceiling", after which the oldest drawings are dropped.

## Docs

- [doc_v4.md](doc_v4.md) - v4 design notes and the v3.1 audit findings
- [doc.md](doc.md) - v3.0 documentation (outdated, kept for history)
