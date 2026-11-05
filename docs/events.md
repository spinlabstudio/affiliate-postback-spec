# Events

All events share an envelope:

| Field | Type | Notes |
|---|---|---|
| `event_id` | string (UUID) | Idempotency key. Same event, same ID, on every retry. |
| `event` | string | One of the names below. |
| `click_id` | string | As received on the landing URL. |
| `player_ref` | string | Pseudonymous operator player ID. Never send email or name. |
| `occurred_at` | string (ISO 8601, UTC) | When it happened, not when it was sent. |
| `brand` | string | Needed when one tracker serves several casino brands. |
| `market` | string (ISO 3166-1 alpha-2) | Deals often differ per country. |

## registration

Sent once, when the account is created. Add `verified: false`; send a separate `kyc_passed` event later if the deal pays only on verified players.

## ftd (first-time deposit)

| Field | Notes |
|---|---|
| `amount` | Decimal string, e.g. `"50.00"`, to avoid float rounding. |
| `currency` | ISO 4217. Crypto deposits: send the fiat value used for accounting as well as the asset. |
| `method` | Optional, e.g. `card`, `bank`, `crypto`. Helps affiliates spot fraud. |

Send FTD only after the deposit is **settled**, not when it is initiated. A declined deposit followed by a successful one is still a single FTD.

## deposit

Optional. Some deals pay on deposit volume or on a qualifying deposit count (for example, CPA after three deposits).

## revenue

A daily batch per player:

| Field | Notes |
|---|---|
| `period` | `YYYY-MM-DD` (UTC day). |
| `ggr` | Gross gaming revenue: stakes minus wins. |
| `bonus_cost` | Bonus money converted to cash or wagered, per your contract definition. |
| `fees` | Payment, platform or game-provider costs if the deal deducts them. |
| `ngr` | The figure the deal is calculated on. State the formula in the contract annex. |
| `restates` | Optional `event_id` of an earlier revenue event this one corrects. |

## chargeback

Sent when a deposit is reversed. Trackers use it to claw back CPA on fraudulent FTDs within the agreed window.
