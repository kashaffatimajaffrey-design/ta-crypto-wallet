# ta-crypto

[![npm version](https://img.shields.io/npm/v/ta-crypto.svg)](https://www.npmjs.com/package/ta-crypto)
[![CI](https://github.com/TDamiao/ta-crypto/actions/workflows/ci.yml/badge.svg)](https://github.com/TDamiao/ta-crypto/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

**Technical analysis for crypto markets — built for production, not tutorials.**

Most TA libraries are TA-Lib ports with a thin npm wrapper. `ta-crypto` is built from the ground up for crypto: session-aware VWAP, funding rate analytics, orderflow proxies, and a streaming API designed for real-time WebSocket feeds — not batch backtests. Every indicator ships with golden tests locked to 1e-10 tolerance and CI-gated compatibility checks against TA-Lib.

---

## Why ta-crypto

**Session-aware VWAP.** Continuous VWAP that never resets is meaningless for crypto. `ta-crypto` lets you define session boundaries explicitly — UTC day, funding window, Asia/US open — so your VWAP reflects how the market actually moves.

**Funding rate analytics.** Cumulative funding and annualized APR out of the box, with presets for 8h, 1h, and 3/day schedules. If you're trading perps, you need this.

**Orderflow proxies.** `signedVolume`, `volumeDelta`, and `orderflowImbalance` give you pressure signals from candle direction and volume — useful when you don't have L2/L3 order book access.

**Streaming-first stateful API.** Feed ticks one at a time. No recomputing from scratch on every price update. Designed for WebSocket loops.

**Signal-driven payout integration.** `examples/flutterwave-signal.ts` shows a complete RSI + volatility regime-gated mobile money disbursement via Flutterwave — HMAC webhook verification, idempotency, and audit logging included.

---

## Install

```bash
npm i ta-crypto
```

---

## Quick start

```ts
import { sma, rsi, macd, bbands, atr, vwapSession, toOHLCV } from "ta-crypto";

const candles = [
  { open: 100, high: 102, low: 99, close: 101, volume: 10, time: 1 },
  { open: 101, high: 103, low: 100, close: 102, volume: 12, time: 2 }
];

const { high, low, close, volume } = toOHLCV(candles, 0);
const s = sma(close, 14);
const r = rsi(close, 14);
const m = macd(close);
const b = bbands(close, 20, 2);
const a = atr(high, low, close, 14);
const vs = vwapSession(high, low, close, volume, [1, 1]);
```

---

## Streaming API

Feed ticks one at a time — no recompute on every update:

```ts
import { createSMA, createEMA, createRSI, createVWAPSession } from "ta-crypto";

const sma14 = createSMA(14);
const ema14 = createEMA(14);
const rsi14 = createRSI(14);

const sma14 = createSMA(14);
sma14.next(101.25);

const vwap = createVWAPSession();
vwap.next({ high: 102, low: 99, close: 101, volume: 10, sessionId: "2026-02-10-asia" });
```

WebSocket loop:

```ts
import { createEMA, createRSI } from "ta-crypto";

const ema21 = createEMA(21);
const rsi14 = createRSI(14);

ws.on("message", (tick) => {
  const price = Number(tick.last);
  const e = ema21.next(price);
  const r = rsi14.next(price);
  if (e !== null && r !== null) {
    // strategy signal path
  }
});
```

---

## Indicators

| Category | Indicators |
|---|---|
| Overlap | `sma` `ema` `rma` `hl2` `hlc3` `ohlc4` `vwap` `bbands` |
| Momentum | `rsi` `macd` `stoch` |
| Volatility | `trueRange` `atr` `natr` `realizedVolatility` |
| Trend | `adx` |
| Volume | `obv` `mfi` |
| Performance | `logReturn` `percentReturn` |
| Crypto-native | `vwapSession` `fundingRateCumulative` `fundingRateAPR` `volatilityRegime` `signedVolume` `volumeDelta` `orderflowImbalance` |

---

## Crypto playbooks

**Session VWAP reset**
- Provide one `sessionId` per candle. VWAP resets exactly when `sessionId` changes.
- Use exchange session boundaries: UTC day, funding window, or custom market session.

**Funding APR**
- `fundingRateAPR(values, periodsPerYear)` annualizes periodic funding.
- `periodsPerYear` presets: `3/day → 1095`, `1/hour → 8760`, `1/8h → 1095`

**Volatility regime**
- `volatilityRegime(values, length, periodsPerYear, lowZ, highZ)` returns `-1` (low), `0` (neutral), `1` (high).
- Default thresholds: `lowZ = -0.5`, `highZ = 0.5`. Increase for noisier low-timeframe pairs.

---

## Accuracy

All indicators tested against TA-Lib and `technicalindicators` in CI. TA-Lib and `technicalindicators` are hard release gates. `pandas-ta` runs as compatibility telemetry.

**External parity** (policy: `scripts/compat-policy.json`):

| Indicator | Burn-in | Tolerance | Blocking refs |
|---|---|---|---|
| SMA(14) | 14 | 1e-10 | TA-Lib, technicalindicators |
| EMA(14) | 14 | 1e-10 | TA-Lib, technicalindicators |
| RSI(14) | 28 | 5e-2 | TA-Lib, technicalindicators |
| MACD(12,26,9) | 80 | 2e-2 | TA-Lib, technicalindicators |
| BBANDS(20,2) | 20 | 1e-10 | TA-Lib, technicalindicators |
| ATR(14) | 56 | 1.5e-1 | TA-Lib, technicalindicators |
| ADX/+DI/-DI(14) | 90 | 1.5 | TA-Lib, technicalindicators |

**Internal parity:**

| Indicator | Reference | Tolerance |
|---|---|---|
| SMA, EMA, RSI | Golden baseline | 1e-10 |
| MACD, BBANDS, ATR, ADX | Golden baseline | 1e-10 |
| VWAP Session | Golden baseline | 1e-10 |
| Stateful RSI(14) | Batch RSI(14) | 1e-10 |

Warmup and alignment: references are left-padded to full series length; comparison starts at indicator-specific burn-in over overlapping non-null windows.

Generate/review vectors:

```bash
npm run generate:golden
npm run test:golden
npm run generate:compat
npm run test:compat:technicalindicators
npm run test:compat:python        # pip install -r scripts/requirements-compat.txt
```

---

## Performance

Benchmarked on 10k candles, Node.js ESM:

```
sma(14): 0.619 ms/run
ema(14): 0.549 ms/run
rsi(14): 0.647 ms/run
macd:    2.887 ms/run
bbands:  2.159 ms/run
atr(14): 1.356 ms/run
adx(14): 5.371 ms/run
```

```bash
npm run bench
```

---

## Examples

| File | What it demonstrates |
|---|---|
| `examples/rsi-strategy.ts` | RSI-based entry/exit logic |
| `examples/vwap-session.ts` | Session VWAP with exchange boundary resets |
| `examples/funding-arbitrage.ts` | Funding rate cumulative + APR |
| `examples/flutterwave-signal.ts` | RSI + volatility regime → signal-gated USDC → M-Pesa KES payout via Flutterwave, with HMAC webhook verification, idempotency, and audit logging |

Using `ta-crypto` in production or experiments? Add your example here.

---

## Typing and inputs

Two input styles across all public APIs:

```ts
// Primitive arrays
rsi([101, 102, 103, 104], 14);

// Candle objects — long keys or aliases (o/h/l/c/v/t)
rsi([{ o: 100, h: 102, l: 99, c: 101, v: 10, t: 1 }], 14);
```

OHLC/OHLCV APIs (`vwap`, `stoch`, `atr`, `natr`, `mfi`, `adx`) accept both array-per-series and candle object formats. Multi-series APIs enforce equal lengths. Numeric inputs are validated as finite numbers with index-aware error messages.

Helpers:

```ts
import { pluckClose, toOHLCV } from "ta-crypto/candles";

const close = pluckClose(candles);
const ohlcv = toOHLCV(candles, 0);
```

---

## Module imports

```ts
import { sma, ema, rsi }               from "ta-crypto/indicators";
import { vwapSession, fundingRateAPR } from "ta-crypto/crypto";
import { toOHLCV, pluckClose }         from "ta-crypto/candles";
import { createSMA, createRSI }        from "ta-crypto/stateful";
```

---

## Contributing

If you use it, improve it. If you miss something, build it. If you're learning, this is a solid codebase to practice financial math.

Start with [CONTRIBUTING.md](./CONTRIBUTING.md) and [ROADMAP.md](./ROADMAP.md). Fastest way in: pick an indicator from the roadmap, implement it, and add tests comparing against TA-Lib.

---

## Release

CI gates every publish — build, tests, and compatibility checks run before anything goes out.

```bash
gh auth login
gh workflow run ci.yml
gh run list --workflow ci.yml --limit 1

npm run changelog
npm run version:patch
npm run release
```

---

## Full API reference and usage guide

See [TUTORIAL.md](./TUTORIAL.md).

---

## License

MIT
