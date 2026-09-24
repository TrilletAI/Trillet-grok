---
name: trillet-phone-mission
description: >-
  Use when the user wants Grok Bot or Cursor to place a real phone call through
  Trillet (restaurant booking, appointment, follow-up, or any outbound voice
  task), including when they have a Trillet account but no phone number yet.
---

# Trillet phone mission

Chat is the UI. Studio is optional. Never invent call outcomes.

## Preconditions (in order)

1. **Trillet MCP connected** — Prefer the live Trillet connector (`trillet_*` tools). If missing or 403, stop and tell the user to connect Trillet (API key + workspace) before continuing. Do not fake a call. Never store API keys in memory, files, or chat.
2. **Destination** — Confirm who to dial (E.164). Never dial without an explicit "call now" / confirm from the user for that destination.
3. **Objective** — One sentence job (for example, book a table for 2 at 7pm Friday). Keep the voice agent prompt short and scoped to that job.

## Number cliff (account exists, no usable from-number)

A usable from-number is an **active** workspace number you can bind to the mission agent (prefer unbound; only rebind an already-bound number after the user explicitly approves taking it from another agent or flow).

If `trillet_list_phone_numbers` shows none usable:

1. Ask country (default from context or user locale: AU / US / GB). For AU, search without forcing `type`; elsewhere ask local vs mobile when needed.
2. `trillet_search_available_numbers` (limit about 5). Show options in a **question widget** (number + type). Never purchase without a pick + explicit yes.
3. After they pick and approve purchase: `trillet_purchase_phone_number` with `confirm: true`.
   - **Free / immediate** — number appears in the workspace; continue.
   - **Stripe Checkout URL** — send the link in chat, tell them to finish payment, then wait for their "paid" / "done" before continuing. Re-list numbers to confirm ownership. Do not bind or dial until the number exists in the list.
4. If purchase fails (address required outside US/CA/AU/GB, stock gone, or similar), explain plainly and re-search or stop.

Optional later: BYO via `trillet_register_external_number` only when the user asks to bring their own number.

## Ensure agent + flow

Reuse a dedicated mission agent and flow when one already exists for this job type in the workspace (for example "Grok Booker"). Otherwise:

1. `trillet_create_agent` (TTS required; a sensible default for AU English is the Rime voice `arcana_eucalyptus`).
2. `trillet_create_call_flow` linked to that agent, `direction: outbound`, prompt = mission only (no scripted greeting that duplicates the welcome).
3. `trillet_set_welcome_message` for outbound.
4. `trillet_bind_phone_number` to that agent (the agent must already have a flow). Label clearly (for example "Grok Booker").

If bind would move a number off another live agent, warn and get explicit approval first.

On a screener or IVR, the prompt must give a clear name and reason, then wait to be connected. Do not hang up at the menu.

## Place the call

1. Summarize: agent, from-number, to-number, one-line script or objective, and that minutes and cost apply.
2. Question widget: Call now / Not yet.
3. On yes: `trillet_start_outbound_call` with `confirm: true`, `agent_id`, `to`, and `from_phone_number_id`.
4. Poll `trillet_get_call` until completed or failed. Report the summary and a short transcript in chat. Never claim success without the call record.

## Hard rules

- No silent purchases, binds that take numbers, or outbound dials.
- Prefer unbound numbers; inventory before creating parallel agents.
- Restaurant booking is one mission. The same ladder works for any outbound objective.
- If MCP or tools error, say what failed and the next real step. Do not roleplay the call in chat as if the phone connected.
