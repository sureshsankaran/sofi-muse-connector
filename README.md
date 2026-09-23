# SoFi Connector (browser-based, for Muse)

SoFi exposes no personal investing API, and Plaid withholds cost basis on SoFi
holdings. This connector drives a live browser through SoFi's website to read
the Active Invest (self-directed brokerage) positions table — including
per-position average cost and cost basis.

**Read-only.** It never places trades, moves money, or changes settings.

## How it works

1. Signs into sofi.com with a saved login from the Secure Vault
   (Cloudflare Turnstile checkbox + SMS/email OTP handled per the standing
   user permissions documented in `SKILL.md`).
2. Opens the Active Invest account's holdings page. The holdings table has no
   cost-basis column, so it opens each position's detail page and reads
   "Avg cost" from the "Your investments" section.
3. Returns a JSON array: `ticker, qty, price, market_value, avg_cost,
   cost_basis, as_of` plus cash rows.

## Layout

- `SKILL.md` — the connector skill (workflow, auth, output contract).
- `references/sofi-notes.md` — live-probe notes on SoFi's login page, bot
  defenses, and layout quirks.

Built for [Muse](https://muse.ai) personal-agent use.
