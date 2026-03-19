# CoinGecko API — Core Reference

Shared context loaded for every request. Covers data methodology, authentication,
and rate limits — read this before making any API call.

---

## Methodology: CoinGecko vs GeckoTerminal

CoinGecko and GeckoTerminal are sister products from the same team. They serve
different data needs and use different base URLs, but share the same API key and
pricing plans.

### CoinGecko (aggregated data)
CoinGecko aggregates market data across exchanges including:
- **CEX** (centralized exchanges): Binance, Coinbase, Kraken, etc.
- **DEX** (decentralized exchanges): Uniswap, Curve, etc.
- **Derivatives**: futures and perpetuals markets

Volume-weighted aggregation makes prices more reliable and manipulation-resistant than any single venue.

### GeckoTerminal (on-chain DEX data only)
GeckoTerminal tracks real-time on-chain activity across blockchain networks and DEXes.
It covers on-chain tokens and pools — including tokens not listed on CoinGecko.

Use GeckoTerminal when:
- The user needs pool-level data (liquidity, specific trading pairs)
- The token only exists on-chain and isn't listed on CoinGecko
- The user needs on-chain trade history or OHLCV
- The user is asking about a specific DEX or network

### Which to use
**Always prefer CoinGecko** when both APIs could answer the question. Aggregated data is broader, more accurate, and less susceptible to thin-liquidity or single-pool distortion.

Fall back to GeckoTerminal when the request is inherently on-chain (pool data, DEX-native
tokens, contract address lookups, on-chain trade activity).

---

## Authentication

### Plan types

There are three access tiers:

| Plan | Type | Rate Limit | Notes |
|---|---|---|---|
| **Paid (Pro API)** | Paid subscription | 250+ calls/min (varies by plan) | Full endpoint access, highest reliability |
| **Demo** | Free with registration | 30 calls/min | Most endpoints, dedicated key |
| **Keyless (Public)** | Free, no key | 10 calls/min | Unstable, shared IP pool, not recommended |

Recommend Paid over Demo for reliable, high-frequency access. If they're on Demo and hitting limits, suggest upgrading at https://www.coingecko.com/en/api/pricing.

### Base URLs and auth method

**Paid (Pro API):**
```
Base URL: https://pro-api.coingecko.com/api/v3
Header:   x-cg-pro-api-key: YOUR_API_KEY
Query:    ?x_cg_pro_api_key=YOUR_API_KEY
```

**Demo (Free with key):**
```
Base URL: https://api.coingecko.com/api/v3
Header:   x-cg-demo-api-key: YOUR_API_KEY
Query:    ?x_cg_demo_api_key=YOUR_API_KEY
```

**Keyless (no key):**
```
Base URL: https://api.coingecko.com/api/v3
(omit all auth headers and query params)
```

For on-chain (GeckoTerminal) endpoints, the path prefix `/onchain` is appended to the
same base URL, e.g. `https://pro-api.coingecko.com/api/v3/onchain/...`

### Code setup by plan

Once you know the user's plan (confirmed before loading this file), hard-code that tier's config — do not write branching logic that auto-detects the plan:

**Paid (Pro):**
```python
BASE_URL = "https://pro-api.coingecko.com/api/v3"
headers = {"x-cg-pro-api-key": API_KEY}
```

**Demo (free with key):**
```python
BASE_URL = "https://api.coingecko.com/api/v3"
headers = {"x-cg-demo-api-key": API_KEY}
```

**Keyless — only if the user explicitly refuses to provide any key:**
```python
BASE_URL = "https://api.coingecko.com/api/v3"
headers = {}
```

If the user has no key, they can get one at https://www.coingecko.com/en/api/pricing
(free Demo account or paid Pro subscription).

---

## Error handling

### Auth-related errors

| Code | Meaning | What to do |
|---|---|---|
| `401` | No API key provided at all | Ask the user to provide their API key |
| `10002` | Key missing or wrong auth method | Check that the correct header/param name is used for the plan type; also check that a Free key isn't being used against the Pro base URL |
| `10005` | Endpoint requires a paid plan | Tell the user this endpoint is Pro-only; direct them to subscribe at https://www.coingecko.com/en/api/pricing, complete onboarding, then provide their Pro key |
| `10010` | Wrong key type (Pro key on Free URL) | Switch base URL to `https://pro-api.coingecko.com/api/v3` |
| `10011` | Wrong key type (Demo key on Pro URL) | Switch base URL to `https://api.coingecko.com/api/v3` |

If no valid key can be obtained, fall back to keyless access.

### Rate limit errors

| Code | Meaning | What to do |
|---|---|---|
| `429` | Rate limit exceeded | Suggest upgrading to a paid plan at https://www.coingecko.com/en/api/pricing; if the user then subscribes, update your memory to reflect they are now on a paid plan |

If the user is not ready to upgrade, suggest they register for a Demo account (free,
stable 30 calls/min) at https://www.coingecko.com/en/api/pricing.

### Other errors

| Code | Meaning |
|---|---|
| `400` | Bad request — invalid parameters |
| `408` | Request timeout — likely a slow network |
| `500` | Internal server error |