# Casino affiliate postback spec

Affiliates in online gambling are paid on what players do after the click: registering, making a first deposit, and generating revenue over months. A **postback** is the server-to-server call that tells the affiliate's tracker that one of those events happened. Pixels in the browser miss too much (ad blockers, app installs, deposits days later), so S2S postbacks are the norm.

Every operator and tracker implements them slightly differently. This spec is an attempt at a sensible default: the events worth sending, the fields each one needs, and the delivery rules that stop double counting. It is vendor-neutral. Use it as a checklist when you integrate a tracker or design your own affiliate module.

## The flow in one picture

```
player clicks affiliate link
  └─> tracker records click, generates click_id
        └─> redirect to casino landing page ?click_id=abc123
              └─> casino stores click_id on the visitor (first-party cookie + sign-up form)
                    └─> player registers  ── postback: registration
                    └─> first deposit     ── postback: ftd
                    └─> daily revenue     ── postback: revenue (batch)
```

## Design principles

1. **The click_id is the join key.** Store it on the player record at registration and never overwrite it. Attribution disputes almost always come from a lost or replaced click ID.
2. **Events are facts, commissions are maths.** Send what happened (amounts, currency, timestamps). Let the tracker or the affiliate system apply CPA, revenue share or hybrid deals.
3. **Every event has an idempotency key.** Retries must never create a second FTD.
4. **Revenue is restatable.** Net gaming revenue changes after chargebacks, bonus cost adjustments and late game-provider reports. Send corrections as new events that reference the period, not as edits.

## Pages

- [Events](events.md): event names and payload fields
- [Delivery and security](delivery.md): retries, ordering, HMAC signing and test mode

## About

Written and kept up to date by Robin Doelman (Spinlab Studio). We build casino platform software with an affiliate system built in, so we have opinions about this. Pull requests from trackers and affiliates are welcome. More of our [longer write-ups on running a casino business](https://spinlab.studio/blog) are on our blog.
