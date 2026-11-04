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

- The ranked pool is **about 300 tokens per timeframe**. Live counts were 317 for `1h`, 310 for `4h` and 301 for `24h`, and they change over time.
- Paging past the end returns **`success: true` with `data: []`**, not an error. A short page (fewer rows than `--page-size`) is the last page. Stop paging there.
- With `--page`/`--page-size`, the whole pool is reachable (e.g. 8 pages × 40). The deprecated `--limit` only reaches the **top 100**.
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
| Platform | `platform` | Array of strings. **Usually `[]`** (~63% of rows), never `null` in practice. Values seen: `pumpfun`, `meteora`, `raydium`, `raydium_launchlab`, `stonkfun_launchpad`, which come from the same enum as `token latest --platform-id`. A row can have several values, e.g. `["raydium_launchlab","raydium"]` for a launchpad token that graduated to a DEX. |

`*_change_pct` compares the current window with the previous window of the same length, in percent (0–100 scale, can go above 100 or below 0). These metrics mean the same as the matching `list-v2` metrics. See [token-list-v2.md](token-list-v2.md#row-fields).

## Ranking and `--timeframe`

- `--timeframe` (`1h` | `4h` | `24h`, default `24h`) chooses **which tokens are listed and in what order**. It does **not** change the metric values. The same token has identical numbers in the `1h` and `24h` lists, and every row includes all four windows.
- The order **doesn't follow any single returned field**. No metric is sorted monotonically. The trending score is opaque, so don't tell the user "ranked by volume". If they want a ranking by a specific metric, use `token list-v2 --sort-by <metric>`.
- The lists overlap only partly: 3 of the top 10 were shared between `1h` and `24h` in one sample.
- **Rankings move within minutes.** The `1h` #1 changed between calls a few minutes apart. Rows can shift, so pages fetched at different times may skip or repeat a token. De-duplicate by `address` when you combine pages.

## Quirks seen in live data

- **Large tokens can appear.** WSOL (`So111…112`) and USDC show up in the pool next to fresh meme tokens. WSOL is the row with `first_listing_time: null`; `token statistic` returns `0` for the same field instead.
- **`*_change_pct` explodes on thin or brand-new tokens.** Values above +10,000% are common, e.g. `volume_4h_change_pct: 53786` on a token whose previous 4h window was nearly empty. Read the percentage next to the absolute metric before calling something a breakout.
- **Mostly micro-caps.** About 96% of rows have 6 decimals (pump.fun-style). Liquidity is often only $10k–$100k and holder counts are in the hundreds. Mention that when you present the list as "what's hot".

## Example

Captured live (`solscan token trending`, default `24h`, Nov 2026). Only the `24h` window and the non-window fields are shown. Real rows have all 85 keys, including the `30m`, `1h` and `4h` windows.

```json
{
  "success": true,
  "data": [
    {
      "address": "FDGdzwKowt4hLkgMFtNik3dL5JeCrvFQqkgxJyGgSG6x",
      "decimals": 6,
      "name": "Hyper Yeti",
      "symbol": "YETI",
      "volume_24h": 147523.6309177736,
      "volume_24h_change_pct": 591.9662737818974,
      "buy_volume_24h": 78067.40028536509,
      "sell_volume_24h": 69456.23063240852,
      "num_trade_24h": 2631,
      "num_buy_trade_24h": 1647,
      "num_sell_trade_24h": 984,
      "num_trader_24h": 631,
      "num_buyer_24h": 577,
      "num_seller_24h": 472,
      "price_24h_change_pct": 447.096548007745,
      "liquidity": 19425.104077933414,
      "market_cap": 56176.178192817824,
      "num_holder": 242,
      "first_listing_time": 1793583938,
      "platform": ["raydium"]
    }
  ],
  "metadata": {}
}
```
