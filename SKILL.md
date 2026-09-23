---
name: "sofi-connector"
description: "Read SoFi Active Invest positions including cost basis via the SoFi website. Use when SoFi holdings or cost basis are needed and Plaid's data is insufficient (Plaid withholds SoFi cost basis). Read-only; never trades."
---

# SoFi Connector (custom, browser-based)

## Purpose
SoFi exposes no personal API and withholds cost basis from Plaid. This skill drives a live browser through SoFi's website to read the Active Invest positions table, including per-position average cost / cost basis. Read-only: it never places trades, transfers money, or changes anything.

## Tooling
- `browser.spawn_task` / `browser.steer_task` for the live session.
- The handoff returns the table as JSON; the parent writes it to a dated raw file such as `sofi-positions-2026-09-23.json` under `~/workspace/goals/portfolio-trade-tracking/hidden_files/`, then merges cost basis into `sofi-portfolio-state.json` per `sofi-basis-playbook.md`.

## Auth
- SoFi website login saved in the Secure Vault (`credentials.request_login` with SoFi's login page). The browser task signs in with the saved login via the normal fill flow.
- SoFi sign-in typically triggers a one-time code (email/SMS). Standing permission covers OTP auto-read: Gmail codes are read automatically; for SMS, detect arrival and ask the user to paste the code or complete the step in the browser. The browser task hands off with `ask_for_information` when it needs a code it cannot obtain itself.
- If no saved login exists, offer browser takeover: the user signs in themselves, then the task continues read-only.

## Workflow
1. Spawn a browser task with a self-contained brief: sign into SoFi with the saved login, dismiss cookie/consent banners, open the Active Invest (self-directed, …1612) account's positions/holdings page.
2. Extract every row: ticker, quantity (including fractional), current price, market value, average cost per share and/or total cost basis, day change if shown.
3. On a 2FA/OTP step, follow Auth above — never invent, reuse, or relay a code outside the approved flow.
4. On a bot challenge/CAPTCHA or login block, stop and hand off `ask_for_information` describing the block (offer user takeover). Do not hammer retries or route around it.
5. Return the table as JSON: `[{"ticker","qty","price","market_value","avg_cost","cost_basis","as_of"}]`.

## Output Contract
- One JSON array, one object per position; tickers bare as shown (no exchange prefix).
- `as_of`: the date shown on the page, else the extraction date (YYYY-MM-DD).
- Include cash/sweep rows (label as shown).
- Write raw output to a dated file (`sofi-positions-YYYY-MM-DD.json`); never overwrite `sofi-portfolio-state.json` directly — merge per the playbook.

## Operating Rules
- Read-only. No orders, no transfers, no settings changes. If any step looks like it would transact, stop and ask.
- SoFi's site layout and bot defenses can break this without warning. On any block, report plainly; do not retry aggressively or route around it.
- SoFi's own cost-basis figures are authoritative for the gains review; Plaid stays the source for intraday balances.
- Credentials live in the Secure Vault only. Never write them to files, logs, or task text.
