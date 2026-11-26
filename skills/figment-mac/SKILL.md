---
name: figment-mac
description: Rent a Mac in the cloud from Figment when a task needs macOS and no Mac is available - an Xcode build or archive, the iOS Simulator, Safari testing, a Mac app, or AppleScript. Covers prices, the agent account, the payment link for the user, ordering, access (SSH and browser console), and how to stop a Mac, start it again and delete it. Use it when the user asks for a Mac, a macOS VM, a cloud Mac or a Mac for an agent, or when a command fails because it needs macOS.
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

Feedback: `POST https://figment.so/api/agent/v1/feedback` with `{"message":"..."}`. No key is needed.
Send it when you are blocked, something that you need is missing, you hit a limit, a document is wrong,
or a task is done. The founder reads each message.

## Before you start

- Rent a Mac only when your user asked for one or agreed to it. Renting costs money.
- Tell your user the plan, the price for an hour or a day, and that setup is not instant.
- If the MCP server `https://figment.so/api/mcp` is connected, use its tools (`figment_get_offer`,
  `figment_create_account`, `figment_get_account`, `figment_buy_credit`, `figment_start_mac`,
  `figment_get_mac`, `figment_get_mac_access`, `figment_stop_mac`, `figment_resume_mac`,
  `figment_delete_mac`, `figment_send_feedback`). `figment_start_mac` orders a new Mac;
  `figment_resume_mac` starts a stopped Mac again. If not, use the HTTP API below.

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
   at `https://figment.so/terms` and allowed the rental. An order also accepts Apple's macOS software
   license agreement (`https://www.apple.com/legal/sla/`) for the account; tell your user.
   - The plan `intel-vm` also needs `"intel_compatibility_accepted":true`; read its notice in the offer
     to your user first.
   - The plans `apple-silicon-vm` and `apple-silicon-vm-8gb` (more memory and storage) also need
     `"allocation_accepted":true`, `"allocation_revision"`, `"power_policy_revision"` and
     `"allocation_disclosure_version"`. Read the plan in the offer. Show its
     `allocation_disclosure.text` to your user first. Then send `allocation_profile.revision`,
     `allocation_profile.power_policy.revision` and `allocation_disclosure.version` of that plan in
     these fields. Each plan has its own values.
5. **Wait.** `GET /macs/{id}` at most once every 30 seconds until `state` is `ready`. Do not promise a
   start time. If the wait is long, tell your user and ask if they want to continue or stop.
6. **Access.** `POST /macs/{id}/access` with `{"console":true,"login":true}`. You get the SSH host,
   port and user name, a one-time link to the Mac screen in a browser, and the Mac login when it is
   available.
7. **Work.** Connect with SSH (`ssh -p <port> <username>@<host>`) and the Mac password, or open the
   console link in a browser. To sign in with a key, sign in one time with the password, add your
   public key to `~/.ssh/authorized_keys` on the Mac, then use the key. An `ssh_public_key` sent with
   the order is not installed on the Mac. The guide has commands that do not print the password
   ("Use SSH from an agent").
8. **Stop.** `POST /macs/{id}/stop` when you pause or the task is done. Billing stops. The Mac and
   your work are kept for seven days. Tell your user the cost from `spent_cents`.
9. **Continue or delete.** To continue, start the same Mac: `POST /macs/{id}/start`, then wait for
   `ready` and read `/access` again. When the Mac is not needed again: `DELETE /macs/{id}`.
10. **Feedback.** `POST /feedback` with `{"message":"<how the task went>"}`. One line helps.

## Rules

- The Mac password and the API key are secrets. Do not print them in chat, logs or commits.
- An hourly Mac bills its first hour when it starts, then by the minute until you stop it, also when it
  is idle. A stop in the first hour costs one hour. Always stop it when the task is done, or tell your
  user that it is still running and what it costs.
- Stop keeps your work. Delete removes it. A stopped Mac is kept for seven days from the stop, at no
  charge, then deleted. Each stop starts a new seven days.
- To continue work, start the stopped Mac again (`POST /macs/{id}/start`). Do not order a new Mac to
  continue your work: a new Mac does not have your files. `can_start` and `start_before` on the Mac
  say if and until when it can start.
- A start is charged when the Mac runs again. Hourly: inside an hour that is paid it costs nothing
  more; after it, a new first hour. Daily: inside the paid day it costs nothing; after it, a new day.
- `DELETE /macs/{id}` deletes the Mac and its data at once. It cannot be undone.
- If an order answers `402 credit_required`, buy credit and send the same order again with the same
  `Idempotency-Key`.
- If an order answers `not_available_now`, tell your user. On `intel-vm`, send `allow_wait: true` only
  if they accept a wait; an hourly Mac bills from the order. `apple-silicon-vm` and
  `apple-silicon-vm-8gb` take no waiting order; try again later.
- Do not promise instant setup, an uptime figure or a refund of unused credit.
- `GET /account` returns `claim.url`. Give it to your user if they want to see the Macs and the credit
  in a browser. It is optional.
- If something stops your task or is missing, tell your user, and send `POST /feedback` with
  `{"category":"blocked","message":"<what happened>"}`. It needs no key. An answer that can stop your
  task has a `feedback_url`. Do not put a key or a password in the text.
- Support: support@figment.so.
