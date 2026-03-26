---
name: coingecko
description: >
  Use this skill for any request involving cryptocurrency market data, coin prices,
  trading volume, market cap, OHLC charts, historical data, exchanges, derivatives,
  NFTs, DeFi, on-chain token data, liquidity pools, DEX data, or anything powered
  by CoinGecko or GeckoTerminal APIs. Trigger this skill whenever the user asks
  about crypto prices, token data, market trends, or wants to call any CoinGecko
  or GeckoTerminal endpoint — even if they don't explicitly say "CoinGecko". Also
  trigger when the user asks about API keys, rate limits, or authentication for
  these APIs.
---

# CoinGecko Skill

You have access to the CoinGecko API (aggregated data) and the GeckoTerminal API
(on-chain DEX data). Together they cover virtually all crypto market data needs.
Both APIs share the same API key and plan tier. GeckoTerminal endpoints use the
same base URL as CoinGecko plus an `/onchain` path prefix (details in `references/core.md`).

## Before anything else

Credentials must come first — base URL and auth header differ by tier, and both key types
start with `CG-` so you cannot infer the plan from the key alone.

**Check memory first.** If the user's plan tier and API key are already saved, confirm they
are still current. Otherwise ask:
1. **Plan tier** — paid (Pro) or free (Demo)?
2. **API key** — their `CG-…` key?

**No key?** Only fall back to keyless access after the user explicitly confirms they have
no key. If the user simply hasn't mentioned a key or tier, ask — do not treat silence as
confirmation. When proceeding keyless, use the Demo base URL with no auth header, but warn
the user it's capped at 5 calls/min and may be unreliable. Suggest a free Demo key at
https://www.coingecko.com/en/api/pricing for more than a few calls.

Then read `references/core.md` for full auth setup and the keyless code pattern, and save
the confirmed plan tier to memory for future sessions.

## Workflow

Once credentials are confirmed, follow this sequence for every request:

1. **Identify the domain** — use the Reference index below to decide which file(s) to load.
2. **Load the relevant reference file(s)** and construct the request.
3. **Execute and handle errors** — auth and rate-limit error codes are documented in
   `references/core.md`. If the API returns error `10005`, the endpoint requires a
   higher plan — inform the user and link them to https://www.coingecko.com/en/api/pricing.
   If you get error `10010` or `10011`, you've used the wrong base URL for the key type —
   swap URLs per `core.md`'s error table and retry automatically.

## Reference index

Load the relevant reference file based on what the user is asking for. You only need to
load the file(s) that match the current request.

### Inspiration & Use cases

If the user asks what they can build, wants project ideas, or asks an exploratory
question like "what data is available?" — load `references/common-use-cases.md` instead
of the domain-specific files below, then follow its pointers to drill deeper.

### CoinGecko (aggregated)

| File | When to load |
|---|---|
| `references/core.md` | **Always read** — auth, methodology, rate limits |
| `references/coins.md` | Coin prices, market data, metadata, tickers, gainers/losers |
| `references/coin-history.md` | Historical charts, OHLC, time-range queries by coin ID |
| `references/coin-supply.md` | Circulating/total supply charts |
| `references/contract.md` | Coin data or charts looked up by token contract address |
| `references/asset-platforms.md` | Blockchain platform IDs, token lists |
| `references/categories.md` | Coin categories and sector market data |
| `references/exchanges.md` | Spot and DEX exchange data, tickers, volume charts |
| `references/derivatives.md` | Derivatives exchanges and tickers |
| `references/treasury.md` | Public company/institution crypto treasury holdings |
| `references/nfts.md` | NFT collection data, market data, charts, tickers |
| `references/global.md` | Global market stats and DeFi data |
| `references/utils.md` | API status, API key usage, supported currencies, search, trending coins/NFTs/categories, exchange rates |

### GeckoTerminal (on-chain)

| File | When to load |
|---|---|
| `references/onchain-networks.md` | Supported networks and DEXes (ID resolution) |
| `references/onchain-pools.md` | Pool discovery, trending/new pools, megafilter |
| `references/onchain-tokens.md` | Token data, price by contract address, holders and traders |
| `references/onchain-ohlcv-trades.md` | OHLCV candles and trade history for pools/tokens |
| `references/onchain-categories.md` | On-chain pool categories (GeckoTerminal-specific) |

## General guidance

- If a request spans multiple domains (e.g. coin price + exchange data), load multiple
  reference files.
- When uncertain which file to load, check the index above before answering.
- For CoinGecko vs GeckoTerminal preference, see `references/core.md` (Methodology section).