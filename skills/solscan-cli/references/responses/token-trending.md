# Token Response Fields — Trending

Field-by-field description of the JSON that `token trending` returns. Read this when you need to interpret or extract fields from the response. For flags, see [../token.md](../token.md#option-details) (`trending` entry). Shared envelope/error shape: [token.md](token.md#common-envelope).

`trending` returns a **ranked list of tokens with identity plus market metrics**: 85 keys per row. Each row has `name`/`symbol`/`decimals`, which `list-v2`/`statistic` rows don't, and a smaller metric set: 4 windows (`30m`, `1h`, `4h`, `24h`) instead of 9, and no supply/FDV/concentration fields.

## Contents

- [Token Response Fields — Trending](#token-response-fields--trending)
  - [Contents](#contents)
  - [Envelope and pagination](#envelope-and-pagination)
  - [Row fields](#row-fields)
  - [Ranking and `--timeframe`](#ranking-and---timeframe)
  - [Quirks seen in live data](#quirks-seen-in-live-data)
  - [Example](#example)

## Envelope and pagination

| Field | Type | Description |
|-------|------|--------------|
| `data` | array | Rows in rank order (rank 1 first). **No `total` field.** |
| `metadata` | object | Envelope-level sibling, `{}` in practice. |

- The ranked pool is **about 250–320 tokens per timeframe** and changes over time. Live counts in Nov 2026 were 264 for `1h`, 262 for `4h` and 248 for `24h` (an earlier sample had 317/310/301).
- Paging past the end returns **`success: true` with `data: []`**, not an error. A short page (fewer rows than `--page-size`) is the last page. Stop paging there.
- With `--page`/`--page-size`, the whole pool is reachable (e.g. 7 pages × 40). The deprecated `--limit` only reaches the **top 100**.
- `--limit N` returns the same rows in the same order as the first N rows of page-based paging.

## Row fields

`<w>` ∈ `30m`, `1h`, `4h`, `24h`. Every row has all 85 keys, whatever `--timeframe` is.

| Group | Fields | Notes |
|-------|--------|-------|
| Identity | `address`, `decimals`, `name`, `symbol` | Name and symbol come from on-chain metadata and **can be spoofed**. One trending token had symbol `SOL` at a `…pump` address. Identify tokens by `address`. |
| Volume (USD) | `volume_<w>`, `buy_volume_<w>`, `sell_volume_<w>` | Each one has a `_change_pct` sibling |
| Trade counts | `num_trade_<w>`, `num_buy_trade_<w>`, `num_sell_trade_<w>` | Each one has a `_change_pct` sibling |
| Unique wallets | `num_trader_<w>`, `num_buyer_<w>`, `num_seller_<w>` | Each one has a `_change_pct` sibling |
| Price change | `price_<w>_change_pct` | Percent, USD-based. **There is no absolute price.** Use `token price-latest` to get one. |
| Market | `liquidity` (USD, summed over all pools), `market_cap` (USD), `num_holder` | |
| Listing | `first_listing_time` | Unix seconds. Usually set, but **`null` for WSOL**. |
| Platform | `platform` | Array of strings. **Usually `[]`** (~60% of rows), never `null` in practice. Values seen: `pumpfun`, `raydium`, `meteora`, `stonkfun_launchpad`, `bags_launchpad`, `raydium_launchlab`, which come from the same enum as `token latest --platform-id`. A row can have several values, e.g. `["stonkfun_launchpad","raydium"]` or `["bags_launchpad","meteora"]` for a launchpad token that graduated to a DEX. |

The API docs mark every metric as nullable ("null when no data is available"). In live data the only `null` seen across the whole pool of all three timeframes was WSOL's `first_listing_time`; all other metrics were numbers. Still treat any metric as possibly `null` before doing math on it. Numbers may come back as integers or floats (e.g. `num_trader_30m_change_pct: 96`).

`*_change_pct` compares the current window with the previous window of the same length, in percent (0–100 scale, can go above 100 or below 0). These metrics mean the same as the matching `list-v2` metrics. See [token-list-v2.md](token-list-v2.md#row-fields).

## Ranking and `--timeframe`

- `--timeframe` (`1h` | `4h` | `24h`, default `24h`) chooses **which tokens are listed and in what order**. It does **not** change the metric values. The same token has identical numbers in the `1h` and `24h` lists, and every row includes all four windows.
- The order **doesn't follow any single returned field**. No metric is sorted monotonically. The trending score is opaque, so don't tell the user "ranked by volume". If they want a ranking by a specific metric, use `token list-v2 --sort-by <metric>`.
- The lists overlap only partly: 3–4 of the top 10 were shared between `1h` and `24h` in samples, while most of the pool (217 of ~250) appeared in both.
- **Rankings move within minutes.** The `1h` #1 changed between calls a few minutes apart. Rows can shift, so pages fetched at different times may skip or repeat a token. De-duplicate by `address` when you combine pages.

## Quirks seen in live data

- **Large tokens can appear.** WSOL (`So111…112`), USDC, USDT, USD1, JUP, PUMP and TRUMP show up in the pool next to fresh meme tokens, sometimes on the first page. WSOL is the row with `first_listing_time: null`; `token statistic` returns `0` for the same field instead.
- **`*_change_pct` explodes on thin or brand-new tokens.** Values above +10,000% are common and outliers reach absurd sizes (e.g. `volume_24h_change_pct: 1156529377884` on a token whose previous 24h window was nearly empty). Values never go below `-100`. Read the percentage next to the absolute metric before calling something a breakout.
- **Mostly small caps.** About 77% of rows have 6 decimals (pump.fun-style); the rest are 9/8-decimal tokens and a few 4/5/7. Median liquidity is about $210k and median holder count about 4k, but many rows sit far below that. Mention it when you present the list as "what's hot".

## Example

Captured live (`solscan token trending`, default `24h`, 10 Nov 2026): the #1 row, with all 85 keys.

```json
{
  "success": true,
  "data": [
    {
      "address": "22JKmY9YZfnQfb3pGYb6hctL5Jyvv3zhL3nQ8oyNuoYp",
      "decimals": 6,
      "name": "Pepetide",
      "symbol": "Pepetide",
      "volume_30m": 27944.683209551316,
      "volume_30m_change_pct": 31.48496376067999,
      "volume_1h": 49197.823044167184,
      "volume_1h_change_pct": -8.517363411036607,
      "volume_4h": 159540.30687142618,
      "volume_4h_change_pct": 54.37319101233234,
      "volume_24h": 3002435.0305047035,
      "volume_24h_change_pct": 679.0484996800832,
      "buy_volume_30m": 17282.284810615703,
      "buy_volume_30m_change_pct": 16.353083726239575,
      "buy_volume_1h": 32135.595317862928,
      "buy_volume_1h_change_pct": 12.481784598416846,
      "buy_volume_4h": 96183.08554080827,
      "buy_volume_4h_change_pct": 86.91256068369869,
      "buy_volume_24h": 1521071.5251504409,
      "buy_volume_24h_change_pct": 633.2037553058512,
      "sell_volume_30m": 10662.398398935613,
      "sell_volume_30m_change_pct": 66.60441792311936,
      "sell_volume_1h": 17062.227726304256,
      "sell_volume_1h_change_pct": -32.31616345514857,
      "sell_volume_4h": 63357.22133061791,
      "sell_volume_4h_change_pct": 22.103115786430266,
      "sell_volume_24h": 1481363.5053542624,
      "sell_volume_24h_change_pct": 732.4970030188167,
      "num_trade_30m": 429,
      "num_trade_30m_change_pct": 23.631123919308358,
      "num_trade_1h": 776,
      "num_trade_1h_change_pct": -26.095238095238095,
      "num_trade_4h": 2933,
      "num_trade_4h_change_pct": 50.179211469534046,
      "num_trade_24h": 48593,
      "num_trade_24h_change_pct": 489.22032254153027,
      "num_buy_trade_30m": 222,
      "num_buy_trade_30m_change_pct": 11.557788944723619,
      "num_buy_trade_1h": 421,
      "num_buy_trade_1h_change_pct": -27.413793103448274,
      "num_buy_trade_4h": 1665,
      "num_buy_trade_4h_change_pct": 63.07541625857003,
      "num_buy_trade_24h": 26439,
      "num_buy_trade_24h_change_pct": 508.4925201380898,
      "num_sell_trade_30m": 207,
      "num_sell_trade_30m_change_pct": 39.86486486486486,
      "num_sell_trade_1h": 355,
      "num_sell_trade_1h_change_pct": -24.46808510638298,
      "num_sell_trade_4h": 1268,
      "num_sell_trade_4h_change_pct": 36.05150214592275,
      "num_sell_trade_24h": 22154,
      "num_sell_trade_24h_change_pct": 467.760123013839,
      "num_trader_30m": 239,
      "num_trader_30m_change_pct": 45.73170731707317,
      "num_trader_1h": 367,
      "num_trader_1h_change_pct": -10.70559610705596,
      "num_trader_4h": 1056,
      "num_trader_4h_change_pct": 20.13651877133106,
      "num_trader_24h": 9637,
      "num_trader_24h_change_pct": 369.18208373904577,
      "num_buyer_30m": 112,
      "num_buyer_30m_change_pct": 38.2716049382716,
      "num_buyer_1h": 180,
      "num_buyer_1h_change_pct": -24.369747899159663,
      "num_buyer_4h": 625,
      "num_buyer_4h_change_pct": 29.132231404958674,
      "num_buyer_24h": 7786,
      "num_buyer_24h_change_pct": 427.1496276235613,
      "num_seller_30m": 148,
      "num_seller_30m_change_pct": 33.33333333333333,
      "num_seller_1h": 240,
      "num_seller_1h_change_pct": -8.045977011494253,
      "num_seller_4h": 688,
      "num_seller_4h_change_pct": 13.71900826446281,
      "num_seller_24h": 7537,
      "num_seller_24h_change_pct": 400.1327140013271,
      "price_30m_change_pct": 14.718895821734279,
      "price_1h_change_pct": 37.354381794303066,
      "price_4h_change_pct": 90.77212205376287,
      "price_24h_change_pct": 286.8638171550943,
      "liquidity": 246544.88082909823,
      "market_cap": 727626.7726979976,
      "num_holder": 3526,
      "first_listing_time": 1794095905,
      "platform": ["raydium"]
    }
  ],
  "metadata": {}
}
```
