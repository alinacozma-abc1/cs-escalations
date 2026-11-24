# Standing policies, reply format details, and tagging

## Bonus swap / conversion requests — no, except Welcome Offer

**Default answer: no**, regardless of usage state. This covers: converting a bonus from one vertical to another (casino↔sport), upgrading to a higher deposit tier, swapping a SuperCoins-shop purchase for a different item, or trading one campaign's reward for another's. "The bonus is completely unused, no harm in swapping it" is **not** the deciding factor — this is a blanket policy based on bonus *type*, not usage.

**The one exception: Welcome Offer bonuses, and only once the corrected/additional deposit has actually happened.** If the bonus in question is specifically a dep1/dep2 Welcome Offer bonus (see the WO sequencing below) and it's completely untouched (`wagered`/`used`/`total_spent`: 0, `state: "new"`), the vertical-swap or higher-tier-upgrade exception is available — but only *after* the player has made the corrected/higher deposit, never speculatively on the strength of an intention to deposit more. If the ticket says the player "wants to" deposit more but hasn't, the answer is: have them make that deposit first, then come back to action the exception.

**Why WO is the only exception:** it's prompted right at registration, before the player has had any real chance to understand what they're picking — a genuine first-touch mistake is plausible there in a way it isn't for a targeted offer the player deliberately opted into. This has been confirmed as the deliberate, permanent policy (not a case-by-case judgment call), including under direct pushback from commercial about extending it to reload/retention campaigns.

## Welcome Offer (WO) sequencing

Greece's sportsbook/casino WO is a two-step deposit chain per vertical: **dep1 → (FS/FC drops) → dep2**. Everything downstream of dep1 — the FS/FC drops and the dep2 deposit campaign itself — is gated on the player actually being in the *segment* populated by qualifying for the real dep1 campaign, not merely holding a dep1-shaped bonus.

- A dep1 bonus can exist in `bondet_state` even when the player never went through the real campaign engine — a CS agent can manually grant it (shows as `campaign_template: "manual_bonus"`, `awarded_by` a staff ID) after a player declined at signup and later changed their mind. A manual grant does **not** populate `customer_attributes.qualified_campaigns` with the real dep1 campaign_id, so nothing downstream will ever trigger for that player even though they "have" the bonus.
- **How to apply a "2nd deposit offer not received" ticket:** check `bondet_state` for the dep1 bonus's `campaign_template` (manual vs. real campaign-triggered), then cross-check `customer_attributes.qualified_campaigns` for the actual dep1 campaign_id. Only escalate as a genuine miss if the player DID qualify for dep1 through the real campaign and dep2 still didn't fire on a qualifying 2nd deposit.
- **A total absence of any WO campaign record (not even a 0%-progress `opted_in`/`eligible` doc) usually means the player declined the offer at registration**, not a targeting miss — a genuine targeting failure typically still leaves behind a partial record. Confirm with the player/CS whether they actually want the offer rather than assuming a bug and granting one.

## No re-grant for a correctly-credited, expired-unused bonus

If a bonus was awarded correctly (right amount, right trigger, matches the campaign) and later expired purely because the player never touched it (`used: 0`, `state_at_expiry: "new"`, no wagering activity in the validity window), do **not** suggest a goodwill re-grant, even as a soft "if the team wants to" aside. State plainly it was credited correctly and expired due to non-use — no manual action needed, full stop. (This is different from a genuinely ambiguous/borderline timing case, like a bet settling seconds after a window closes — those can still warrant a goodwill mention; see the settlement pattern in `known-bugs-and-patterns.md`.)

## Qualifying after the campaign window already closed — no award by default

If a player's qualifying action (a deposit, a bet) happens **after** the live campaign window had already closed, the default is **no award**. The only exception is a demonstrably **longer window communicated to that specific player** than the system actually enforced — evidence, in order of strength: the ticket itself quoting specific communicated terms ("the offer says available for 5 days"), the bonus's public T&C page explicitly stating a longer deadline, or an accessible comms record to that player. The *absence* of a stated deadline in the T&C is not itself an exception — it just means there's no evidence either way, and the default holds. Check this per ticket even when a similar-looking prior ticket got a goodwill exception — don't inherit the exception from pattern-matching to that prior case without re-verifying it for this specific player and campaign instance.

## Zero enrollment is not automatically the known enrollment-write bug

A player having zero `customer_campaign_status` record for a deposit-triggered offer has at least two very different causes that look identical: (a) a genuine backend bug where a player deposits within the real promo window but the enrollment never gets written, or (b) the player simply deposited after the campaign/communicated window had already closed, so there was never anything to enroll into. Only (a) deserves a GWG award. Before recommending one: read any screenshot/comm date in the ticket carefully (don't assume the month from skimming), find the specific campaign instance's actual removal/expiry timestamp, and compare it directly to the player's deposit timestamp. Only call it the known bug if the deposit falls genuinely close to (within hours/a day of) the real window's close and still produced zero record.

## A missing one-time attribute means the action didn't happen

When a `customer_attributes` field tied to a specific one-time player action (a promo code entered at signup, an opt-in flag) is simply absent, the default read is **the player didn't do that action** — not that a tracking/acquisition pipeline silently failed. Don't jump to "our tracking is broken" without separate, positive evidence of a pipeline failure (e.g. multiple players independently known to have used the same code/flow all missing the same attribute, or a known incident). A single player's missing attribute, on its own, is not that evidence.

## "Can't place bet with this bonus," no obvious config reason — re-award directly

For "can't use this bonus" tickets: first check the bonus definition for a real restriction (odds minimum, market whitelist, a wagering conflict with another active bonus) and explain it if found (no re-award, direct the player to the correct bet type). If nothing in the config explains the rejection, **recommend re-awarding directly** rather than first chasing a screenshot/error message — for a low-value, often same-day-expiring free bet, the back-and-forth costs more than the bonus is worth, and a consistent player-reported symptom (tried multiple clients, same result) is enough to act on. Only escalate for engineering follow-up if the same symptom recurs across multiple players on the same bonus/campaign — that points to an actual platform bug worth fixing, not a one-off.

## Reply format — the details

- **Open addressed to the CS agent who filed the ticket, by first name** (from the Zendesk "X in Ticket #N" line) — never open with "@crm-ops-greece checked #N..."; the tag is not the reply's audience.
- **Always include an explicit award/no-award recommendation** when the investigation supports one. Presenting findings without a verdict, when the evidence points somewhere, is an incomplete reply.
- **VIP goodwill rule:** for VIP players who narrowly missed an opt-in/qualification window by a small margin (minutes, not hours/days) — e.g. opting into a cashback/promo a few minutes after the qualifying activity — the standing policy is to generally recommend paying the compensation as a goodwill gesture, even when the system was technically correct to withhold it automatically. Don't stop at "this is a timing miss, not a bug" for a VIP without following up with an explicit goodwill recommendation, unless there's a clear reason not to (repeated abuse pattern, missed by a lot more than a few minutes).
- **A campaign-wide wave of narrow, rule-driven misses can itself be the signal of a misconfiguration**, worth a wider goodwill call regardless of VIP status — e.g. several tickets same day hitting one campaign with stakes slightly over/under a single exact-match threshold where a `>=` was clearly intended instead of `==`. If several tickets cluster this way, flag the possible misconfiguration explicitly rather than just quietly noting "narrow miss, maybe lenient?" on each one individually.
- **If the ticket has no real issue description** (just a campaign name + ticket code, nothing else), don't infer the implied question and answer a guess — ask the CS agent what specifically needs checking.
- **Tag(s) go at the end of the message**, never the opening.
- **Don't manually sign a "Sent using Claude" line** — the Slack integration auto-appends this itself.

## Tagging — the full decision table

| Condition | Tag(s) |
|---|---|
| The reply recommends an **award**, OR flags that additional checks/follow-up are needed from the team, OR surfaces a campaign-level problem pattern for the **first time** that day/thread (a bug, display issue, paused/broken campaign affecting multiple players) | `<!subteam^S0AAXR6SJKU>` (@crm-ops-greece) |
| The reward/mechanic is **Supercoins-denominated** anywhere in the reward or the dispute — missions paying out in Supercoins, a SuperCoins shop purchase/redemption, a Supercoins prize from the SuperSpin wheel (regular FS/FC/cash wheel prizes are NOT this — crm-ops-greece) | `<!subteam^S0BGAAABDPU>` (@loyalty-crm-ops) **instead of** crm-ops-greece |
| Ticket text, Topic field, or the cited campaign name mentions **"Serial Winner"** or **"Super Streak"** (same product — Super Streak is the internal name) — fixtures, display errors, contested streaks | `<!subteam^S0CCR3M59RR>` (@hunch-cs, replaced @hunch-team on 2026-11-12), **stacked on top of** whichever of the above also applies |
| **Basketaki MVP**, or any physical-item/loyalty-shop-merchandise fulfillment question | `<@U0AH4BR70P6>` (George Papaspyros — a person mention, not a subteam) |
| **Casino tournament** scoring/points tickets (e.g. Diamond Challenger — not a Betler campaign; Diamond Challenger is a stake counter: points from wagers, not wins) | `<!subteam^S0BJUUP3RQT>` (tournament owners), not crm-ops-greece |
| A **plain no-award with nothing for anyone to action** | **No tag at all.** Don't default to crm-ops-greece "just in case" — a repeat of an already-flagged pattern, with a no-award verdict and nothing new to do, gets no tag. |

**Mention syntax:** usergroup/subteam IDs (the ones starting with `S`) need the `<!subteam^ID>` form — `<@ID>` is for individual user mentions and does **not** render as a clickable tag for a usergroup ID, it just shows as literal text. George Papaspyros's ID (`U...`) is a real user, so `<@U0AH4BR70P6>` is correct for him specifically.

**Checking which tag applies is not a one-time pattern-match.** The Supercoins carve-out in particular is the single most-repeated correction in this workflow — a ticket filed under an ordinary-looking Zendesk Topic ("Sports::Bonus Not Granted") can still be a Supercoins mission underneath. Check the actual campaign name/reward type for mission/coins/Supercoins signals every time, independent of the Zendesk category it happened to be filed under. Don't infer a subteam ID by copying whatever `<!subteam^...>` a teammate happens to use nearby in the same thread for their own unrelated reason — always use the literal IDs in this table.

## A note on automating this routine

Don't set up an unattended recurring job (cron, scheduled task) to run this triage routine by itself. Two independent failure modes have been confirmed: (1) the underlying Betler data connection needs periodic manual re-authentication that an unattended run can't do, and (2) even when the data connection is fine, sending the actual Slack reply is a write action that may need interactive approval — an unattended run has been observed to successfully react 👀 on new tickets (a lower-risk action) while silently sending zero replies all night, which is arguably worse than no automation: a ticket with only a 👀 reaction and no reply reads as "someone's on it" when no one actually is. Run this routine live, with someone watching, each time it's needed.
