# Token Response Fields — Markets

Field-by-field description of the JSON the `token markets` command returns. Read this when you need to interpret or extract specific fields from a response, not when you're just building the command (see [../token.md](../token.md) for flags/params). Shared envelope/error shape: [token.md](token.md#common-envelope).

This covers **DEX pool/market listings** for a token or token pair — liquidity and trading-activity stats per pool — as opposed to a wallet- or token-scoped activity log ([token-activity.md](token-activity.md)).

## Contents

- [`markets`](#markets)

> Only actions with a confirmed field-level source are documented here. If the action you need isn't listed yet, fall back to the `--no-json` output or `--help`, and treat unlabeled fields at face value rather than guessing their meaning.

## `markets`

`solscan token markets --token <TOKEN_OR_PAIR> [--sort-by <field>] [--program <addresses>] [--page <n>] [--page-size <n>]`

`--token` takes 1 or 2 comma-separated addresses (3+ → `400 "token" must contain less than or equal to 2 items`): **1 token** returns every pool where that mint is either side; **2 tokens** scopes to pools that are exactly that pair — the result is identical whichever order you pass them in. `data` is an **array** — one object per pool, always **descending** by `--sort-by` (`volume`\|`trade`\|`tvl`\|`trader`). Omitting `--sort-by` gives the same order as `--sort-by volume`. There is no `--sort-order` for this action (unlike `token defi`/`defi-export`) and no `total`/`has_next` metadata — page by incrementing `--page` until a response comes back with fewer than `--page-size` rows (an exhausted page returns `data: []`, `metadata: {}`).

Top-level fields (in addition to the [common envelope](token.md#common-envelope)):

| Field | Type | Description |
|-------|------|--------------|
| `data` | array of object | One object per pool — see the table below. |
| `metadata` | object | Not in Solscan's published API reference but present on every live response. Holds `tokens`; is `{}` when `data` is empty. |
| `metadata.tokens` | object | Map keyed by mint address → token identity, covering **exactly** the mints that appear as `token_1`/`token_2` in this page's `data` (no more, no less). Use it to label pools without a separate `token meta-multi` call. |
| `metadata.tokens.<mint>.token_address` | string | Same mint address as the key. |
| `metadata.tokens.<mint>.token_name` | string | Token name (e.g. `Wrapped SOL`). |
| `metadata.tokens.<mint>.token_symbol` | string | Token symbol (e.g. `WSOL`, `USDC`). |
| `metadata.tokens.<mint>.token_icon` | string | Icon URL. **Absent** for many tokens (≈40% of mints in sampled pages) — not an empty string. |

Each item in `data`:

| Field | Type | Description |
|-------|------|--------------|
| `pool_id` | string | The pool/AMM account address — the on-chain account representing this specific market. |
| `program_id` | string | Address of the DEX/AMM program that owns the pool (e.g. Raydium, Whirlpool, PumpSwap). |
| `token_1` | string | Mint address of one side of the pool. |
| `token_2` | string | Mint address of the other side of the pool. |
| `token_account_1` | string | The pool's own token account (vault) holding its `token_1` reserves — not a user wallet's token account. |
| `token_account_2` | string | The pool's own token account (vault) holding its `token_2` reserves. |
| `total_trades_24h` | integer | Number of trades against this pool in the last 24h. |
| `total_trades_prev_24h` | integer | Number of trades in the prior 24h window (i.e. 24–48h ago), for computing 24h-over-24h change. Can be **absent** — see interpretation tips. |
| `total_volume_24h` | number | Trading volume in the last 24h, in USD. |
| `total_volume_prev_24h` | number | Trading volume in the prior 24h window, in USD. Can be **absent**. |
| `total_tvl` | number | Total value locked in the pool, in USD — what `--sort-by tvl` sorts on. Can be **absent**, even on major pools (e.g. a SOL/USDC pool with $38M 24h volume), and can be a near-zero value like `1.1e-06` — see interpretation tips. |
| `num_trader_24h` | integer | Count of unique trader addresses in the last 24h — what `--sort-by trader` sorts on. |
| `num_trader_prev_24h` | integer | Unique trader count in the prior 24h window. Can be **absent**. |

No field is ever `null` — missing data is expressed by omitting the key.

**Example** (`solscan token markets --token So11111111111111111111111111111111111111112`, live call on 2026-11-10; trimmed to 3 of 10 rows — row 1 is a typical pool, row 2 lacks `total_tvl`, row 3 has a near-zero `total_tvl` — and `metadata.tokens` trimmed to the mints in those rows)

```json
{
  "success": true,
  "data": [
    {
      "pool_id": "8FnX3xo2yYw3EUE6w3nQA4GfXGS9wpK6oj3veJpbFzLo",
      "program_id": "BiSoNHVpsVZW2F7rx2eQ59yQwKxzU5NvBcmKshCSUypi",
      "token_1": "So11111111111111111111111111111111111111112",
      "token_2": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
      "token_account_1": "ATRsNGv2nDw7hSMfkUTBoVUDsFDwN7po7KbecyiGWNB4",
      "token_account_2": "2Y7HATmn9aJBcxCskE5V2U2epmjvkZmB51zTJBbhj4cU",
      "total_trades_24h": 740264,
      "total_trades_prev_24h": 789378,
      "total_volume_24h": 110866338.76650375,
      "total_volume_prev_24h": 113172732.90193667,
      "total_tvl": 4333877.246802122,
      "num_trader_24h": 87354,
      "num_trader_prev_24h": 90374
    },
    {
      "pool_id": "FnhxUP3dcQbypCUmGWw55ijxPBxifPT558UQSCYDfcCU",
      "program_id": "SCoRcH8c2dpjvcJD6FiPbCSQyQgu3PcUAWj2Xxx3mqn",
      "token_1": "So11111111111111111111111111111111111111112",
      "token_2": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
      "token_account_1": "44GW6aFire4Fd72h4QqjenNKrQLjfnApHhWYXo3S1gvp",
      "token_account_2": "34WpFzQ1WE2nDLLEKJuvCXxapL7ak6CigJS8Ks5NDo5K",
      "total_trades_24h": 146448,
      "total_trades_prev_24h": 128407,
      "total_volume_24h": 38355735.15547554,
      "total_volume_prev_24h": 37862275.03620668,
      "num_trader_24h": 18914,
      "num_trader_prev_24h": 17452
    },
    {
      "pool_id": "AAt6dwREhNfa3BvoLJWeSDjwVmV2jqHwXB3sB3ERynE8",
      "program_id": "pAMMBay6oceH9fJKBRHGP5D4bD4sWpmSwMn52FMfXEA",
      "token_1": "So11111111111111111111111111111111111111112",
      "token_2": "DMvcPrWsThg3eyM96jEFTqFYrVSEV7jVzU5HEggRdTzP",
      "token_account_1": "GXGPTi63RBYa9EtzGpTDT7RMaP8cN2KmW3Ddht5SuhkV",
      "token_account_2": "3Gj19jy7r9jf81FDjcReV5shA3YuzSFAeMebrPj6AqmN",
      "total_trades_24h": 82032,
      "total_trades_prev_24h": 11228,
      "total_volume_24h": 73715015.65139328,
      "total_volume_prev_24h": 10277737.222983321,
      "total_tvl": 1.165511737718596e-06,
      "num_trader_24h": 4373,
      "num_trader_prev_24h": 1506
    }
  ],
  "metadata": {
    "tokens": {
      "So11111111111111111111111111111111111111112": {
        "token_address": "So11111111111111111111111111111111111111112",
        "token_name": "Wrapped SOL",
        "token_symbol": "WSOL",
        "token_icon": "https://raw.githubusercontent.com/solana-labs/token-list/main/assets/mainnet/So11111111111111111111111111111111111111112/logo.png"
      },
      "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v": {
        "token_address": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
        "token_name": "USDC",
        "token_symbol": "USDC",
        "token_icon": "https://cdn.rwa.xyz/images/asset-icons/128/color/usdc.png"
      },
      "DMvcPrWsThg3eyM96jEFTqFYrVSEV7jVzU5HEggRdTzP": {
        "token_address": "DMvcPrWsThg3eyM96jEFTqFYrVSEV7jVzU5HEggRdTzP",
        "token_name": "Extended",
        "token_symbol": "XTEN",
        "token_icon": "https://ipfs.io/ipfs/bafkreida2jrucir7m54eyb3d4xyfv6mjqii4hbt3q7xp7rjxojgt634h6m"
      }
    }
  }
}
```

**Interpretation tips**

- **Optional fields** — `total_tvl` and the three `_prev_24h` fields (`total_trades_prev_24h`, `total_volume_prev_24h`, `num_trader_prev_24h`) are omitted on some rows rather than set to `0`/`null`. The `_prev_24h` trio is missing together (on 62 of 100 SOL pools in one sample, mostly newer/low-activity pools); `total_tvl` is missing independently. Treat a missing field as "no data", not as zero, and check it exists before computing a percent change or summing TVL.
- **Near-zero `total_tvl`** — many pools, especially PumpSwap (`pAMMBay6…`) pools, report a `total_tvl` well below $1 (e.g. `1.1e-06`) despite millions in 24h volume. Solscan likely can't price the non-SOL side. Don't read it as "empty pool". To rank real liquidity venues, use `--sort-by tvl`, which puts well-priced pools first; rows without `total_tvl` don't appear at the top of that sort.
- **Sort is single-key** — results are ordered only by the `--sort-by` metric. Other metrics in the same page aren't ordered (e.g. with `--sort-by trade`, `total_volume_24h` is not monotonic).
- **Token order is the pool's own** — `token_1`/`token_2` follow the pool's internal ordering, not the order you passed to `--token`. SOL is `token_1` in most SOL pools but `token_2` in others (e.g. `BAft…/So11…`). Check both fields.
- **Names and symbols** — read them from `metadata.tokens[<mint>]` instead of calling `token meta`. Fall back gracefully when `token_icon` is missing.
- **Vaults, not wallets** — `token_account_1`/`token_account_2` are the pool's vaults, not any user's wallet token account. Don't confuse them with the `token_account` fields returned by `account tokens`.
- **No spot price** — this endpoint reports pool-level aggregates (volume, trades, TVL, traders), not a USD price per token. For a token's USD price, use `token price-latest`/`price-history`.
- **Unknown address** — a valid address with no pools (e.g. a wallet address) returns `200` with `data: []` and `metadata: {}`, not an error. A malformed address returns `400 Address [...] is invalid`. `--program` addresses go through the same validation.
- **Paging** — this action has no CSV/export twin, and no page cap was observed (`--page 10000` with SOL still returned rows). Rare transient `502`s happen on deep pages; retry before concluding the data ran out.
