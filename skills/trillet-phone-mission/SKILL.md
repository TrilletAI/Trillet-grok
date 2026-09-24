---
name: Trillet phone mission
description: >-
  Use when the user wants Grok Bot to place a real phone call through Trillet
  (restaurant booking, appointment, follow-up, personal update, or any outbound
  voice task), including when they have a Trillet account but no phone number
  yet.
---
# Trillet phone mission

Chat is the UI. Studio is optional. Never invent call outcomes.

## Preconditions (in order)

1. **Trillet MCP connected** — Prefer the live Trillet connector (`trillet_*` tools). If missing or 403, stop and tell the user to connect Trillet (API key + workspace) before continuing. Do not fake a call.
2. **Destination** — Confirm who to dial (E.164). Never dial without an explicit “call now” / confirm from the user for that destination.
3. **Objective** — One sentence job (e.g. book table for 2 at 7pm Friday at X, or “tell X that Sebastian is fine and picking up Anna”). Keep the voice agent prompt short and scoped to that job.

## Number cliff (account exists, no usable from-number)

A usable from-number is an **active** workspace number you can bind to the mission agent (prefer unbound; only rebind an already-bound number after the user explicitly approves stealing it from another agent/flow).

If `trillet_list_phone_numbers` shows none usable:

1. Ask country (default from context or user locale: AU / US / GB). For AU, search without forcing `type`; elsewhere ask local vs mobile when needed.
2. `trillet_search_available_numbers` (limit ~5). Show options in a **question widget** (number + type). Never purchase without a pick + explicit yes.
3. After they pick and approve purchase: `trillet_purchase_phone_number` with `confirm: true`.
   - **Free / immediate** — number appears in the workspace; continue.
   - **Stripe Checkout URL** — send the link in chat, tell them to finish payment, then wait for their “paid” / “done” before continuing. Re-list numbers to confirm ownership. Do not bind or dial until the number exists in the list.
4. If purchase fails (address required outside US/CA/AU/GB, stock gone, etc.), explain plainly and re-search or stop.

Optional later: BYO via `trillet_register_external_number` only when the user asks to bring their own number.

## Ensure agent + flow

Reuse a dedicated mission agent/flow when one already exists for this job type in the workspace (e.g. “Grok Booker”). Otherwise:

1. `trillet_create_agent` (TTS required; sensible default Rime `arcana_eucalyptus` for AU English; human-like voice off unless the user asks).
2. `trillet_create_call_flow` linked to that agent, `direction: outbound`, prompt = mission only (no scripted greeting that duplicates the welcome).
3. `trillet_set_welcome_message` for outbound (short custom message).
4. Apply **Ad-hoc call defaults** below (required every time you create or reuse an ad-hoc/mission flow for Grok calls).
5. `trillet_bind_phone_number` to that agent (agent must already have a flow). Label clearly (e.g. “Grok Booker”).

If bind would move a number off another live agent, warn and get explicit approval first.

## Ad-hoc call defaults (always)

Before every outbound dial on a Grok/ad-hoc mission flow, ensure these are set. Saying “bye” in speech does **not** hang up the SIP leg — the runtime needs end-call + voicemail settings.

Via `trillet_set_call_settings`:

- `voicemail_detection`: `{ enabled: true, action: "leave_message", amdDetection: true }`
- `end_call_on_silence`: `10`
- `silence_reminder_retries`: `1` (keep reminders minimal — “are you there?” on voicemail is bad UX)
- `max_call_duration`: `180` for short personal/mission calls (raise only when the mission needs longer)
- `max_call_duration_message`: short goodbye
- `customer_location`: user timezone (default `Australia/Melbourne` when unknown AU)
- `time_context_enabled`: `true`

Via `trillet_update_call_flow` settings:

- `enableEndCallInstructions`: `true`
- `enableHumanLikeVoiceAndTone`: `false` unless the user asked for expressive voice

Every mission prompt must include: after the job is done (or after one voicemail message), goodbye and **immediately use the end-call action**; do not ask “are you there?” after a voicemail; do not speak again after the update.

## Place the call

1. Summarize: agent, from-number, to-number, one-line script/objective, that minutes/cost apply.
2. Question widget: Call now / Not yet — skip the widget only when the user already gave an explicit dial-now for that number and message in the same turn.
3. On yes: `trillet_start_outbound_call` with `confirm: true`, `agent_id`, `to`, `from_phone_number_id`.
4. Poll `trillet_get_call` (optionally `include_function_calls: true` if hangup looks wrong) until completed/failed. Report summary + short transcript in chat. Never claim success without the call record.

## Hard rules

- No silent purchases, binds that steal numbers, or outbound dials.
- Prefer unbound numbers; inventory before creating parallel agents.
- Always apply Ad-hoc call defaults before dialing a Grok mission agent.
- Restaurant booking is one mission template; same ladder works for any outbound objective.
- If MCP/tools error, say what failed and the next real step — do not roleplay the call in chat as if the phone connected.
