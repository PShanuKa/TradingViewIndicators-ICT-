# ICT Suite v4.0 — Documentation

> **File:** `ICT_v4.pine` | **Platform:** TradingView (Pine Script v5)
> **Status:** v3.1 හි P0/P1/P2 issues සියල්ල නිවැරදි කළ full rewrite එකකි.
> v3.1 (`ICT_v3.1.pine`) reference එකක් ලෙස repo එකේ තබාගෙන ඇත.

---

## 1. v3.1 → v4.0: කුමක් වෙනස් වුණාද

### 1.1 Statistics engine (P0 — වැදගත්ම)

| # | v3.1 හි ප්‍රශ්නය | v4.0 හි විසඳුම |
|---|---|---|
| 1 | එකම bar එකේ SL+TP දෙකම වැදුනොත් **හැම විටම WIN** | `Same-bar SL+TP resolution` input — default **Conservative (SL first)**. ඒ bars ගණන dashboard එකේ පෙන්වයි |
| 2 | Trailing R = `floor(high excursion)` (කල්පිත අගයක්) | ඇත්ත trailing stop එකේ **exit price** එකෙන් R ගණනය කරයි |
| 3 | Fixed සහ Trailing එකම stop එකක් බෙදාගත්තා | **Simulation දෙකක්** සම්පූර්ණයෙන් වෙන් වෙන්ව — Fixed එකට BE/trail කිසිසේත් නැත |
| 4 | Break-even exit = Loss | **Win / Break-even / Loss** ලෙස 3-way |
| 5 | Entry bar එකේම BE/exit trigger වුණා | `bar_index > entryBar` — entry bar එක සම්පූර්ණයෙන් skip |
| 6 | Chart signals ≠ tracked signals | Label ඇඳෙන්නේ track වන trade වලට පමණි. Skip වූ ඒවා වෙනම counter වල |
| 7 | Spread/slippage නැහැ | `spreadTicks` + `slipTicks` හැම trade එකකින්ම R එකෙන් අඩු වේ |
| — | Profit factor / Max DD / Expectancy නැහැ | දෙකටම වෙන වෙනම report වේ |

### 1.2 Signal logic (P1)

| # | v3.1 | v4.0 |
|---|---|---|
| 8 | Liquidity sweep detect වුණේ EQH/EQL හැදුණු bar එකේ පමණි (ප්‍රායෝගිකව කවදාවත් නෑ) | Untapped liquidity pool array එකක්, **හැම confirmed bar එකකම** scan වේ |
| 9 | Trend + Premium/Discount filters signal engine එකෙන් අයින් වෙලා | දෙකම **CORE** එකට ආපහු (toggle කළ හැක) |
| 10 | HTF bias = පසුගිය 1H candle එකේ පාට | HTF **EMA + candle** (mode selectable), 4H default |
| 11 | `htfH[2]` → LTF bars 2ක් පස්සට (වැරදි) | `high[3]` **HTF series** එකේ → නිවැරදි HTF FVG |
| 12 | London KZ = 02:00–05:00 **UTC** (පැය 5ක වැරැද්දක්) | Session times `America/New_York` timezone එකේ → **DST automatic** |
| 13 | Zone එකට wick එකක් ඇල්ලුවත් entry, SL එක ATR only | Zone tap + rejection close; **SL = zone edge − ATR buffer** |
| 14 | එකම zone එකෙන් නැවත නැවත signals | Zone-level `traded` flag — zone එකකට **එක signal** පමණයි |
| 15 | Dynamic TP කවදාවත් minRR එකට වඩා ලං target එකක් ගත්තේ නෑ | ඇත්ත liquidity target එක ගණන් කර, R:R එක අඩු නම් **trade එක skip** කරයි |

### 1.3 ICT depth (P2)

| # | v3.1 | v4.0 |
|---|---|---|
| 16 | OB = `close > highest(10)` breakout detector | OB = **displacement candle** (body > ATR×) + ඒ move එකෙන් **FVG** එකක් හැදීම + ඊට කලින් තිබූ opposite candle |
| 17 | නවතම pivot එක BOS reference → minor pivot break = false BOS | **Unbroken reference level** එකක් per side; uptrend එකක minor pullback high ignore වේ |
| 18 | හැම pivot එකකටම IDM line | **එක IDM level** per side, swept වූ පසු නැවත set වේ; entry filter එකක් ලෙසත් භාවිත කළ හැක |
| 19 | OTE නැහැ | OTE 0.62–0.79 band + confluence condition |
| 20 | Breaker = color වෙනස් කිරීම පමණි | Breaker = direction flip + `traded` reset (අලුත් setup එකක්) |

### 1.4 Infrastructure (P3)

- `alert()` (dynamic: entry/SL/TP/R) + `alertcondition()` — webhook automation සඳහා
- සියලුම state mutations `barstate.isconfirmed` යටතේ → **realtime/historical divergence නැත**
- Drawing object pools — පරණ ඒවා age-based delete (500 limit එකට ගැහෙන්නේ නැත)
- Zone/liquidity arrays age + count capped → performance predictable

---

## 2. Entry Model

```
CORE (ඔක්කොම සපුරන්න ඕන — එකින් එක off කළ හැක)
  1. Untraded, valid zone එකකට price tap වීම (OB / FVG / Breaker)
  2. Enabled killzone එකක් ඇතුළත
  3. ATR volatility healthy
  4. Market structure aligned          (trend == +1 buy / −1 sell)
  5. Discount (BUY) / Premium (SELL)
  6. Inducement swept                  (optional)

CONFLUENCE (5න් minConf ගණනක් ඕන — default 2)
  X  HTF bias aligned
  Y  Opposing liquidity swept (sweepWindow ඇතුළත)
  Z  Recent BOS / MSS in direction
  W  Price inside OTE band (0.62–0.79)
  V  Zone stack — OB + FVG overlap

RISK GATE
  ⇢ SL, TP, R:R ගණනය → R:R < minRR නම් trade එක SKIP (count වේ)
```

Signal එකේ label එකේ tooltip එකේ මේ ඔක්කොම, entry/SL/TP/R සමග පෙන්වයි.

---

## 3. Dashboard — column දෙකේ අර්ථය

| Section | කියන දේ |
|---|---|
| **MARKET** | Structure, HTF bias, killzone, volatility, range (Prem/Disc), OTE, live zones, swept liquidity |
| **LIVE SETUP** | දැන් BUY/SELL core pass ද, confluence කීයද, projected R:R කීයද, position එකේ තත්වය |
| **PERFORMANCE** | **FIXED TP** vs **TRAILING** — trades, W/BE/L, win rate, net R (costs අඩු කර), expectancy, profit factor, max DD, best/worst |
| **ENGINE** | Skip වූ bars (low R:R / in trade), same-bar SL+TP bars, cost per trade |

> **ENGINE section එක ඉතාම වැදගත්.** "Same-bar SL+TP bars" ගණන ලොකු නම් — ඔබේ SL/TP timeframe එකට වඩා ලංයි, එවිට backtest එකේ නිරවද්‍යතාවය අඩුයි. "Skipped: low R:R" ලොකු නම් — `minRR` අඩු කරන්න හෝ SL එක තව tight කරන්න.

---

## 4. නිර්දේශිත ආරම්භක Settings (XAUUSD 15m)

| Input | අගය | හේතුව |
|---|---|---|
| Swing Pivot Length | 10 | 15m එකට සමතුලිතයි |
| Displacement: body > ATR × | 1.0 | මෙය ඉහළ දැම්මොත් OB අඩුයි, quality වැඩියි |
| Require Displacement to leave an FVG | ✅ | නියම ICT OB එකේ අත්‍යවශ්‍ය කොටස |
| Min FVG size (ATR ×) | 0.25 | Noise gaps ඉවතලයි |
| Killzone Timezone | America/New_York | **වෙනස් කරන්න එපා** |
| Trade London / NY Open KZ | ✅ / ✅ | Asian + London Close default off |
| HTF | 240 (4H) | 15m entry එකකට 1H ලංයි; 4H bias පැහැදිලියි |
| Min Confluence | 2 | 3 → අඩු signals වැඩි quality; 1 → වැඩි signals |
| SL Anchor | Structure | Zone invalidation එකයි ඇත්ත SL එක |
| TP Mode | Liquidity | Untapped pool එකට යවයි |
| Minimum R:R | 1.5 | + `Skip if target < Min R:R` ✅ |
| Spread (ticks) | 20 | ඔබේ broker එකේ ඇත්ත gold spread එක දාන්න |
| Slippage per side | 5 | 15m market order එකකට යථාර්ථවාදීයි |
| Time stop (bars) | 120 | 15m → දින 1.25ක් පමණ |

---

## 5. Testing Protocol — v3.1 එකට වඩා හොඳද කියලා දැනගන්නේ කොහොමද

v3.1 හි ප්‍රධානම ප්‍රශ්නය තමයි **measure කරන ක්‍රමයම වැරදි වීම**. ඒ නිසා v4 valid ද කියලා තහවුරු කරන්න මේ පිළිවෙළ අනුගමනය කරන්න:

1. **Baseline:** `ICT_v4.pine` chart එකට දාලා, `Count stats from` = මාස 3කට කලින් date එකක් දාන්න.
2. **Fixed column එක විතරක් බලන්න** මුලින්. Trailing column එක comparison එකක් මිසක් target එකක් නෙවෙයි.
3. මේ 3 numbers ලියාගන්න: **Trades**, **Expectancy / trade**, **Profit factor**.
   - Expectancy > 0 නම් edge එකක් තියෙනවා
   - Profit factor > 1.3 නම් ප්‍රායෝගිකව tradeable
   - Trades < 20 නම් **තීරණයක් ගන්න එපා** — sample එක කුඩායි
4. **එක් input එකක් පමණක්** වෙනස් කර නැවත බලන්න (`Min Confluence` 2→3 වගේ). එකවර කිහිපයක් වෙනස් කළොත් කුමක් වැඩ කළාද කියලා දැනගන්න බැහැ.
5. **Same-bar SL+TP bars** ගණන trades ගණනෙන් 20%ට වඩා වැඩි නම් → ප්‍රතිඵල විශ්වාස කරන්න එපා, ලොකු timeframe එකකට යන්න හෝ SL පළල් කරන්න.
6. Instrument 2–3ක (XAUUSD, EURUSD, NAS100) same settings වලින් test කරන්න. එකක් විතරක් හොඳ නම් ඒක curve-fit එකක්.

> ⚠️ **Forward test** එකක් නැතුව live යන්න එපා. Indicator එකේ tracker එක historical bars මත ගණනය කරන one-trade-at-a-time simulation එකක් — ඒක strategy backtester එකක් නෙවෙයි.

---

## 6. දැනට තියෙන සීමා (හිතාමතාම)

| සීමාව | හේතුව / ඊළඟ පියවර |
|---|---|
| **Intrabar order නොදනී** | SL/TP එකම bar එකේ නම් input එකෙන් තීරණය වේ. නිවැරදිම විසඳුම 1m data එකෙන් `request.security_lower_tf()` — v4.1 සඳහා |
| **Partial TP (TP1/TP2) නැහැ** | Dual-sim එකට 3වන column එකක් ලෙස v4.1 හි එකතු කළ හැක |
| **`strategy()` version නැහැ** | Equity curve, position sizing, commission model සඳහා `ICT_v4_strategy.pine` වෙනම port එකක් ඕන |
| **News filter නැහැ** | Pine එකෙන් economic calendar access නැහැ. Manual session blackout input එකක් දාන්න පුළුවන් |
| **එකවර එක trade එකයි** | `Allow new signal while a trade is open` on කළොත් signals පෙන්වයි, නමුත් tracker එක තාම single-slot |
| **Skipped counters bar-based** | එකම setup එක bars කිහිපයක් skip වුණොත් කිහිප වතාවක් count වේ — coverage indicator එකක් මිසක් exact setup ගණනක් නෙවෙයි |

---

## 7. Alerts

| ක්‍රමය | භාවිතය |
|---|---|
| `alert()` (dynamic) | "Any alert() function call" තෝරන්න. Message එකේ **entry, SL, TP, R** ඇතුළත් — webhook/bot automation සඳහා සූදානම් |
| `alertcondition()` | "ICT v4 — BUY signal" / "SELL signal" — සරල notification සඳහා |
| Exit alerts | `Alert on Exit` on නම් W/BE/L සහ R value සමග |

---

*ICT Suite v4.0 — Structure Engine + Dual Simulation. v3.1 audit එකේ P0–P3 items සියල්ල ආවරණය කර ඇත.*
