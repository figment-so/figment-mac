---
name: figment-mac
description: Rent a Mac in the cloud from Figment when a task needs macOS and no Mac is available - an Xcode build or archive, the iOS Simulator, Safari testing, a Mac app, or AppleScript. Covers prices, the agent account, the payment link for the user, ordering, access (SSH and browser console) and stopping the Mac. Use it when the user asks for a Mac, a macOS VM, a cloud Mac or a Mac for an agent, or when a command fails because it needs macOS.
license: MIT
metadata:
  author: Figment
  homepage: https://figment.so/agents.md
---

# Rent a Mac from Figment

Figment rents Macs by the hour or by the day through an HTTP API and an MCP server. No sign-in is needed.
Your user pays one Stripe link for prepaid credit, and you spend that credit.

The full guide is `https://figment.so/agents.md`. Read it when you need a detail that is not here.
Live plans, prices and availability are at `https://figment.so/api/agent/v1/offer`. Always read prices
from there. Do not quote a price from memory.

## Before you start

- Rent a Mac only when your user asked for one or agreed to it. Renting costs money.
- Tell your user the plan, the price for an hour or a day, and that setup is not instant.
- If the MCP server `https://figment.so/api/mcp` is connected, use its tools (`figment_get_offer`,
  `figment_create_account`, `figment_get_account`, `figment_buy_credit`, `figment_start_mac`,
  `figment_get_mac`, `figment_get_mac_access`, `figment_stop_mac`). If not, use the HTTP API below.

## Steps

API base: `https://figment.so/api/agent/v1`. Send JSON. After step 2, send
`Authorization: Bearer $FIGMENT_API_KEY` on every call.

1. **Prices.** `GET /offer`. Choose a plan id. Check `available_now`.
2. **Account.** If `FIGMENT_API_KEY` is not set: `POST /accounts` with `{"agent_name":"<your name>"}`.
   Save `api_key` where it stays secret (an environment variable or a secret store, never a commit).
   Make one account and use it again. Do not make an account for each task.
3. **Credit.** `GET /account`. If `credit_cents` is too low: `POST /credit` with
   `{"amount_cents": <cents>}` and an `Idempotency-Key` header. Give `checkout_url` to your user and ask
   them to pay. If you have a Link wallet, you can pay the link after your user approves. Read
   `GET /account` again until `credit_cents` rises. Do not say that the payment is complete before that.
4. **Order.** `POST /macs` with
   `{"plan":"<plan id>","period":"hourly","accept_terms":true,"authorization":"user_authorized"}` and a
   new `Idempotency-Key`. Send `accept_terms` and `authorization` only if your user agreed to the terms
   at `https://figment.so/terms` and allowed the rental. The plan `intel-vm` also needs
   `"intel_compatibility_accepted":true`; read its notice in the offer to your user first.
5. **Wait.** `GET /macs/{id}` at most once every 30 seconds until `state` is `ready`. Do not promise a
   start time. If the wait is long, tell your user and ask if they want to continue or stop.
6. **Access.** `POST /macs/{id}/access` with `{"console":true,"login":true}`. You get the SSH host and
   port, a one-time link to the Mac screen in a browser, and the Mac login when it is available.
7. **Work.** Connect with SSH (`ssh -p <port> <username>@<host>`) or open the console link in a browser.
8. **Stop.** `POST /macs/{id}/stop` when the task is done. Billing stops. Tell your user the cost from
   `spent_cents`.

## Rules

- The Mac password and the API key are secrets. Do not print them in chat, logs or commits.
- An hourly Mac bills by the minute from the order until you stop it, also when it is idle. Always stop
  it when the task is done, or tell your user that it is still running and what it costs.
- A stopped Mac cannot be started again. Copy the files you need before you stop it. Data is kept for
  seven days after a stop, then deleted.
- If an order answers `402 credit_required`, buy credit and send the same order again with the same
  `Idempotency-Key`.
- If an order answers `not_available_now`, tell your user. Send `allow_wait: true` only if they accept
  a wait; an hourly Mac bills from the order.
- Do not promise instant setup, an uptime figure or a refund of unused credit.
- `GET /account` returns `claim.url`. Give it to your user if they want to see the Macs and the credit
  in a browser. It is optional.
- Support: support@figment.so.
