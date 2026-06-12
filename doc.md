# ICT Concepts v3 — Indicator Documentation

> **භාෂාව:** Pine Script v5 | **Platform:** TradingView  
> **Version:** 3.0 — Entry Signal + Win Rate + Trailing R:R System  
> **File:** `ICT_v3.pine`

---

## Table of Contents

1. [Overview](#1-overview)
2. [Chart Visual Elements](#2-chart-visual-elements)
   - [Order Blocks (OB)](#21-order-blocks-ob)
   - [Fair Value Gaps (FVG)](#22-fair-value-gaps-fvg)
   - [Market Structure — BOS / MSS](#23-market-structure--bos--mss)
   - [Liquidity Zones — EQH / EQL](#24-liquidity-zones--eqh--eql)
   - [Premium & Discount Zones](#25-premium--discount-zones)
   - [Killzones / Sessions](#26-killzones--sessions)
   - [Breaker Blocks (BB)](#27-breaker-blocks-bb)
   - [Inducement (IDM)](#28-inducement-idm)
   - [HTF Confluence](#29-htf-confluence)
3. [Entry Signal Engine](#3-entry-signal-engine)
   - [BUY Conditions](#buy-conditions-6-points)
   - [SELL Conditions](#sell-conditions-6-points)
   - [Signal Label & SL/TP Lines](#signal-label--sltp-lines)
4. [Win Rate Tracker](#4-win-rate-tracker)
5. [Trailing R:R Simulator](#5-trailing-rr-simulator)
6. [Dashboard — Full Field Reference](#6-dashboard--full-field-reference)
   - [Signals Section](#signals-section)
   - [Win Rate Section](#win-rate-section)
   - [R:R Report Section](#rr-report-section)
   - [Market Section](#market-section)
7. [Settings / Inputs Reference](#7-settings--inputs-reference)

---

## 1. Overview

**ICT Concepts v3** යනු Inner Circle Trader (ICT) methodology ඇසුරෙන් ගොඩනගා ඇති TradingView Indicator එකකි. මෙය Market structure, Order flow, සහ Price delivery zones හඳුනාගනිමින් ස්වයංක්‍රීය Entry Signals ලබා දෙයි. ඒ සමගම ලබාදෙන trade outcome track කරමින් Win Rate, Streak statistics, සහ Trailing R:R comparison report Dashboard එකක් ලෙස Chart එකේ display කරයි.

**Key features:**
- ✅ Order Blocks (Bullish & Bearish) + Breaker Block conversion
- ✅ Fair Value Gaps (auto-filled detection)
- ✅ Market Structure — BOS & MSS labels
- ✅ Liquidity Zones — Equal Highs (EQH) & Equal Lows (EQL)
- ✅ Premium / Discount / Equilibrium zones
- ✅ Killzone session highlights (Asian, London, NY)
- ✅ Inducement (IDM) levels
- ✅ HTF Confluence (Higher Timeframe OB / FVG)
- ✅ Entry Signal Engine (6-condition scoring system)
- ✅ Win Rate Tracker with Streak statistics
- ✅ Unlimited Trailing R:R Simulator

---

## 2. Chart Visual Elements

### 2.1 Order Blocks (OB)

**කුමක්ද:** Order Block යනු Strong Move එකකට පෙර ඇති Candle(s) නිර්මාණය කළ Supply/Demand zone එකකි.

| Visual | Meaning |
|--------|---------|
| 🟦 Blue box (semi-transparent) | Bullish OB — Price ඉහළ break කර ගිය zone |
| 🟧 Orange/Red box (semi-transparent) | Bearish OB — Price පහළ break කර ගිය zone |

**Detection logic:**
- **Bullish OB:** `close > highest_high[lookback]` AND previous candle bearish (`close[1] < open[1]`)
- **Bearish OB:** `close < lowest_low[lookback]` AND previous candle bullish (`close[1] > open[1]`)

**Mitigation:** OB midpoint (`(top + bottom) / 2`) touch වූ විට OB mitigated ලෙස mark වේ. Breaker Block ON නම් box color වෙනස් වේ, නැතිනම් remove වේ.

---

### 2.2 Fair Value Gaps (FVG)

**කුමක්ද:** Candle 3ක් අතර price overlap නොවන gap zone එකකි — liquidity imbalance.

| Visual | Meaning |
|--------|---------|
| 🟩 Green box | Bullish FVG — `low[1] > high[3]` |
| 🟥 Red box | Bearish FVG — `high[1] < low[3]` |

**Auto-fill:** Price gap zone fill කළ විට box automatically remove වේ.  
**Extend:** "Extend Until Filled" ON නම් box current bar දක්වා extend වේ.

---

### 2.3 Market Structure — BOS / MSS

**BOS (Break of Structure):** Trend continuation — previous swing high/low break.  
**MSS (Market Structure Shift):** Trend reversal — opposite direction break.

| Label | Color | Meaning |
|-------|-------|---------|
| `BOS` | 🔵 Blue | Structure continuation — same trend |
| `MSS` | 🟠 Orange | Trend reversal signal |
| ⚪ Circle above bar | Swing High point |
| ⚪ Circle below bar | Swing Low point |

**Dashed line:** Broken swing level සිට current bar දක්වා horizontal line.

---

### 2.4 Liquidity Zones — EQH / EQL

**කුමක්ද:** ඒකිත Highs (EQH) හෝ ඒකිත Lows (EQL) — stop losses cluster වන zones.

| Label | Visual | Meaning |
|-------|--------|---------|
| `EQH` | 🩷 Pink dotted line | Equal Highs — Buy stops cluster |
| `EQL` | 🟣 Purple dotted line | Equal Lows — Sell stops cluster |

**Sweep detection:** EQH/EQL swept (broke) වූ පසු **10 bars** ඇතුළත Signal engine හට EQH/EQL sweep condition ලෙස report කරයි.

**Threshold:** Two pivots `≤ liqThresh %` apart නම් Equal levels ලෙස සලකයි (default 0.05%).

---

### 2.5 Premium & Discount Zones

**කුමක්ද:** Recent range (default 50 bars) හි 50% equilibrium based zones.

| Zone | Color | Meaning |
|------|-------|---------|
| 🟥 Premium (upper half) | Red | Sellers' zone — SELL setups |
| 🟩 Discount (lower half) | Green | Buyers' zone — BUY setups |
| `EQ 50%` dashed line | Yellow | Equilibrium — Fair price |

Labels: `Premium`, `Discount`, `EQ 50%` chart right edge හි visible වේ.

---

### 2.6 Killzones / Sessions

Chart background color change කරමින් high-probability trading windows highlight කරයි.

| Session | Background | UTC Time | Used in Signals |
|---------|-----------|----------|-----------------|
| Asian | 🩶 Grey | 20:00 – 00:00 | ❌ No |
| London Open | 🔵 Indigo | 02:00 – 05:00 | ✅ Yes |
| New York Open | 🟦 Teal | 07:00 – 10:00 | ✅ Yes |

> **Note:** Asian session signal logic සඳහා excluded — low probability zone ලෙස සලකයි.

---

### 2.7 Breaker Blocks (BB)

**කුමක්ද:** Mitigated OB role reverse වී opposite direction zone ලෙස act කිරීම.

| Visual | Meaning |
|--------|---------|
| 🟢 Green box (was bearish OB) | Bullish Breaker — was Bearish OB, now support |
| 🔴 Red box (was bullish OB) | Bearish Breaker — was Bullish OB, now resistance |

OB mitigate වූ විට box color/border automatically switch වේ.

---

### 2.8 Inducement (IDM)

**කුමක්ද:** Main swing levels හි ඇතුළත ඇති minor pivot points — stop hunt targets.

| Visual | Meaning |
|--------|---------|
| `IDM` dotted line | Minor pivot below main swing high (or above swing low) |

**Condition:** IDM pivot < last swing high (bullish) හෝ IDM pivot > last swing low (bearish).

---

### 2.9 HTF Confluence

**කුමක්ද:** User-selected Higher Timeframe (default: 60min) හි OB/FVG zones.

| Label | Visual | Meaning |
|-------|--------|---------|
| `HTF OB` | 🟦/🟧 large box | HTF Order Block zone |
| `HTF FVG` | Semi-transparent box | HTF Fair Value Gap |

Signal Engine හි `C6` condition ලෙස HTF Bias (bull/bear) use කරයි.

---

## 3. Entry Signal Engine

6-condition scoring system. Min conditions (default: 4/6) meet වූ විට signal fire වේ.

### BUY Conditions (6 points)

| Code | Condition | Details |
|------|-----------|---------|
| C1 | **Bullish Trend** | Market structure bullish (`trend == 1`) |
| C2 | **Discount Zone** | Price below 50% equilibrium |
| C3 | **Killzone Active** | London or NY session |
| C4 | **Bull OB or Bull FVG** | Price inside unmitigated bullish zone |
| C5 | **EQL Swept** | Equal Lows swept within last 10 bars |
| C6 | **HTF Bull Bias** | Higher TF candle bullish (`htfClose > htfOpen`) |

### SELL Conditions (6 points)

| Code | Condition | Details |
|------|-----------|---------|
| C1 | **Bearish Trend** | Market structure bearish (`trend == -1`) |
| C2 | **Premium Zone** | Price above 50% equilibrium |
| C3 | **Killzone Active** | London or NY session |
| C4 | **Bear OB or Bear FVG** | Price inside unmitigated bearish zone |
| C5 | **EQH Swept** | Equal Highs swept within last 10 bars |
| C6 | **HTF Bear Bias** | Higher TF candle bearish (`htfClose < htfOpen`) |

**Duplicate prevention:** Same zone හි duplicate signal fire නොවන ලෙස flag control ක්‍රමය use කරයි.

### Signal Label & SL/TP Lines

| Visual | Meaning |
|--------|---------|
| 🟢 `BUY 5/6` label below bar | BUY signal, score 5 out of 6 |
| 🔴 `SELL 4/6` label above bar | SELL signal, score 4 out of 6 |
| 🔴 Dashed `SL` line | Stop Loss = `low - (ATR × SL_Multiplier)` |
| 🟢 Dashed `TP` line | Take Profit = `entry + (entry - SL) × RR_Ratio` |
| 🟢 `W` circle | Trade reached TP — WIN |
| 🔴 `L` circle | Trade reached SL — LOSS |

**Tooltip (hover on label):**
```
BUY SETUP | Score: 5/6
[OK] Trend: Bullish
[OK] Zone: Discount
[OK] Session: Killzone
[OK] Zone: In Bull OB/FVG
[OK] Liquidity: EQL Swept
[--] HTF: No Bias
```

---

## 4. Win Rate Tracker

Signal fire වූ පසු TP හෝ SL කුමක් hit වේදැයි track කරයි. **Confirmed bars** use — repainting නැත.

**State Machine:**
```
IDLE (0) → LONG pending (1) or SHORT pending (-1) → WIN or LOSS → IDLE (0)
```

| Tracked Variable | Meaning |
|------------------|---------|
| `totalSignals` | Total BUY/SELL signals fired |
| `totalWins` | Fixed TP hit trades |
| `totalLosses` | SL hit trades |
| `currentWinStreak` | දැනට consecutive wins |
| `maxWinStreak` | All-time maximum consecutive wins |
| `currentLossStreak` | දැනට consecutive losses |
| `maxLossStreak` | All-time maximum consecutive losses |

---

## 5. Trailing R:R Simulator

**Concept:** Fixed TP close කිරීම වෙනුවට SL step-up (trailing) කළොත් ලැබෙන R comparison.

**Logic:**
```
Price hits 1R → SL moves to Break-Even (0R locked)
Price hits 2R → SL moves to +1R (1R locked)
Price hits 3R → SL moves to +2R (2R locked)
... unlimited
```

**Trade closed at SL:**
- 0R locked → Loss = **-1R** (same as fixed)
- 1R locked → Result = **0R** (Break-Even)
- 2R locked → Result = **+1R** (partial profit)

**Trade closed at TP:**
- Trailing R = `max(lockedR, rrRatio)` — fixed TP level or better

| Tracked Variable | Meaning |
|------------------|---------|
| `tradeEntryPrice` | Entry price of current open trade |
| `tradeRUnit` | 1R in price (= entry − SL distance) |
| `trailingLockedR` | Current bars locked R level |
| `totalRFixed` | Cumulative R — fixed TP method |
| `totalRTrailing` | Cumulative R — trailing method |
| `bestTradeR` | Single best trade R achieved |
| `trailWins` | Trailing method: trades closed at BE or better |
| `trailLosses` | Trailing method: trades closed at -1R |

---

## 6. Dashboard — Full Field Reference

Chart corner හි display වන table (position configurable).

### Signals Section

| Field | Meaning |
|-------|---------|
| **BUY Score** | Current bar BUY conditions met count (e.g. `BUY: 5/6`) |
| **SELL Score** | Current bar SELL conditions met count (e.g. `SELL: 3/6`) |
| **Active Trade** | `LONG Active` / `SHORT Active` / `No Trade` |

### Win Rate Section

| Field | Meaning | Color Logic |
|-------|---------|-------------|
| **Total Signals** | All signals fired so far | White |
| **Wins** | Fixed TP hit count | Green |
| **Losses** | SL hit count | Red |
| **Win Rate** | `(Wins / Signals) × 100` | Green ≥60% / Yellow ≥45% / Red <45% |
| **Win Streak (Max)** | `current (all-time max)` — e.g. `3 (7)` | Green |
| **Loss Streak (Max)** | `current (all-time max)` — e.g. `2 (4)` | Red |

### R:R Report Section

| Field | Meaning | Color Logic |
|-------|---------|-------------|
| **Fixed TP Total R** | Total R earned/lost via fixed TP | Green if positive / Red if negative |
| **Trailing Total R** | Total R earned/lost via trailing SL | Bright green / Red |
| **Trailing Advantage** | `Trailing R − Fixed R` difference | Cyan if positive / Orange if negative |
| **Avg R (Trailing)** | Average R per trade (trailing method) | White |
| **Best Trade R** | Single best trade — max R achieved | Yellow |
| **Trail Win Rate** | Win% where BE or better = "win" | Green ≥60% / Yellow ≥45% / Red <45% |
| **Live Locked R** | Open trade — R currently locked in | Bright green / `—` if no trade |

### Market Section

| Field | Meaning | Color Logic |
|-------|---------|-------------|
| **Structure** | `Bullish` / `Bearish` / `Neutral` | Green / Red / Grey |
| **Session** | `NY Open` / `London` / `Asian` / `Off Hours` | White |
| **Active OBs / FVGs** | Count of unmitigated OBs and open FVGs | Cyan |

---

## 7. Settings / Inputs Reference

### Order Blocks
| Setting | Default | Description |
|---------|---------|-------------|
| Show Order Blocks | ✅ On | OB boxes on/off |
| OB Lookback (bars) | 10 | Swing high/low detection range |
| Bullish OB Color | Cyan (75% transparent) | Bull OB fill color |
| Bearish OB Color | Red-Orange (75% transparent) | Bear OB fill color |

### Fair Value Gaps
| Setting | Default | Description |
|---------|---------|-------------|
| Show FVGs | ✅ On | FVG boxes on/off |
| Extend Until Filled | ✅ On | Box extends to current bar until filled |

### Market Structure
| Setting | Default | Description |
|---------|---------|-------------|
| Show BOS / MSS | ✅ On | Structure labels on/off |
| Swing Length | 10 | Pivot detection length |
| Show Dashed Line | ✅ On | Horizontal line at broken level |

### Liquidity Zones
| Setting | Default | Description |
|---------|---------|-------------|
| Show Liquidity Zones | ✅ On | EQH/EQL on/off |
| Pivot Length | 10 | Pivot lookback |
| Equal Level Threshold % | 0.05 | Max % difference for "equal" levels |

### Premium & Discount
| Setting | Default | Description |
|---------|---------|-------------|
| Show Premium/Discount | ✅ On | Zone boxes on/off |
| Range Lookback | 50 | Bars to calculate range high/low |

### Killzones
| Setting | Default | Description |
|---------|---------|-------------|
| Show Killzones | ✅ On | Session backgrounds on/off |

### Breaker Blocks
| Setting | Default | Description |
|---------|---------|-------------|
| Show Breaker Blocks | ✅ On | Mitigated OB color switch |

### Inducement (IDM)
| Setting | Default | Description |
|---------|---------|-------------|
| Show Inducement Levels | ✅ On | IDM lines on/off |
| IDM Pivot Length | 5 | Minor pivot detection length |

### HTF Confluence
| Setting | Default | Description |
|---------|---------|-------------|
| Show HTF OB/FVG | ❌ Off | HTF zones on/off |
| Higher Timeframe | 60 (1H) | Timeframe to pull HTF data |

### Entry Signals
| Setting | Default | Description |
|---------|---------|-------------|
| Show Entry Signals | ✅ On | BUY/SELL labels on/off |
| Min Conditions to Fire | 4 | 3–6 conditions needed for signal |
| Take Profit RR Ratio | 2.0 | TP = Entry + (SL distance × RR) |
| SL ATR Multiplier | 1.5 | SL = `low - (ATR(14) × multiplier)` |
| Show SL/TP Lines | ✅ On | SL/TP dashed lines on/off |

### Dashboard
| Setting | Default | Description |
|---------|---------|-------------|
| Show Dashboard | ✅ On | Dashboard table on/off |
| Position | Bottom Right | Top Right / Top Left / Bottom Right / Bottom Left |

---

## Quick Reference — Chart Labels Summary

| Label / Shape | Location | Meaning |
|---------------|----------|---------|
| `BUY n/6` | Below bar | BUY signal fired, n conditions met |
| `SELL n/6` | Above bar | SELL signal fired, n conditions met |
| `W` circle | Near bar | Trade WIN (TP hit) |
| `L` circle | Near bar | Trade LOSS (SL hit) |
| `BOS` | At swing level | Break of Structure |
| `MSS` | At swing level | Market Structure Shift |
| `EQH` | At equal high | Equal Highs — stop cluster |
| `EQL` | At equal low | Equal Lows — stop cluster |
| `IDM` | Minor pivot | Inducement level |
| `HTF OB` | Large box | Higher TF Order Block |
| `HTF FVG` | Large box | Higher TF Fair Value Gap |
| `SL` | Dashed red line | Stop Loss level |
| `TP` | Dashed green line | Take Profit level |
| `Premium` | Right edge | Price above 50% range |
| `Discount` | Right edge | Price below 50% range |
| `EQ 50%` | Right edge | Range midpoint |

---

*Last updated: ICT Concepts v3.0 — Entry Signal + Win Rate + Unlimited R:R Trailing System*
