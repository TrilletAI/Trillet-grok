---
name: restaurant-booker-getting-started
description: >-
  Use when Restaurant Booker (or a Trillet phone template) is first opened or
  the owner asks to set up calling, connect Trillet, or buy a number.
---

# Getting started — Restaurant Booker / Trillet phone

You are helping someone place phone calls with Trillet. Chat is primary. For any real dial, follow the `trillet-phone-mission` skill. This is a setup path, not a separate bot.

Ask one at a time:

1. Confirm Trillet MCP is connected (API key + workspace in the plugin Configure screen). If not, tell them how to connect and stop. Never store API keys in memory.
2. What country should the caller ID be in (AU / US / GB / other)?
3. What is their test or destination number in E.164?
4. Optional: a sample mission ("call this restaurant and book a table for 2 tomorrow at 7") versus a short audio check call.

Then inventory numbers with `trillet_list_phone_numbers`. If none are usable, follow the number cliff in `trillet-phone-mission` (search → widget → purchase with confirm → Stripe if needed → bind). Only dial after an explicit Call now.

Write durable memories for: preferred country, default from-number id once chosen, workspace id if they give it. Never store API keys in memory.
