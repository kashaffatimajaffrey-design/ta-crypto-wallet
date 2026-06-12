# ta-crypto

[![npm version](https://img.shields.io/npm/v/ta-crypto.svg)](https://www.npmjs.com/package/ta-crypto)
[![CI](https://github.com/TDamiao/ta-crypto/actions/workflows/ci.yml/badge.svg)](https://github.com/TDamiao/ta-crypto/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Technical analysis for crypto markets — built for real-time feeds, not backtests.

---

`ta-crypto` ships the indicators you actually need for crypto: session-aware VWAP that resets on your terms, funding rate analytics for perp traders, orderflow proxies when you don't have L2 data, and a streaming stateful API you can wire directly into a WebSocket loop. Every indicator is golden-tested to 1e-10 tolerance and CI-gated against TA-Lib.

```bash
npm i ta-crypto
```

---

## What's inside

**Classic indicators** — SMA, EMA, RSI, MACD, Bollinger Bands, ATR, ADX, VWAP, and more.

**Crypto-native** — session VWAP with explicit boundary resets, funding rate cumulative + APR, volatility regime labeling, signed volume, volume delta, orderflow imbalance.

**Streaming API** — stateful `createRSI`, `createEMA`, `createVWAPSession` and friends. Feed one tick at a time, no recompute.

**Production examples** — including a signal-gated mobile money payout via Flutterwave: RSI + volatility regime fires a USDC → M-Pesa KES transfer with HMAC webhook verification and audit logging. See `examples/flutterwave-signal.ts`.

---

## Full docs

See [TUTORIAL.md](./TUTORIAL.md) for the complete API reference, input formats, compatibility matrix, and benchmarks.

---

## Contributing

If you use it, improve it. Start with [CONTRIBUTING.md](./CONTRIBUTING.md) and [ROADMAP.md](./ROADMAP.md).

---

MIT
