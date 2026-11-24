---
name: ce-crm-gr-ticket-triage
description: Investigate and answer CRM OPS complaint tickets in Superbet Greece's #ce-crm-gr Slack channel — player disputes about bonuses, free spins, missions, Supercoins, deposits, and promo codes not being granted, credited, or counted. Use this whenever you're handling a #ce-crm-gr ticket, checking a player's bonus/campaign status in Betler, deciding whether to recommend an award or goodwill credit, or figuring out who to tag on a reply (crm-ops-greece vs loyalty-crm-ops vs hunch-team vs the physical-prize contact) — even if the request doesn't say "ticket" explicitly, e.g. "check this player's Road to 10 mission", "why didn't this bonus credit", "can we swap this bonus", "is this a known bug". Also covers the standing reply format and award/no-award decision rules so answers are consistent across whoever is handling the queue.
---

# CRM OPS ticket triage for #ce-crm-gr (Superbet Greece)

This channel gets a steady stream of tickets from CS agents on behalf of players, almost all some version of "player says X happened/didn't happen with a bonus/mission/deposit, can you check Betler?" The job is: investigate in Betler (and sometimes Notion/Snowflake for campaign identity), form an actual verdict, and reply in-thread in a consistent format that tells the right team what to do next.

The single most important habit this skill encodes: **a ticket that "looks like" a pattern you've seen before still needs its own verification.** Almost every mistake documented in the reference files below is the same shape — matching a ticket to a known pattern (a prior bug, a prior ruling, a campaign you recognize) and skipping the specific check that would have confirmed or ruled it out for *this* player, *this* bet, *this* date. Read a ticket's specific numbers before reasoning from precedent.

## Workflow

1. **Identify the ticket.** It's a Zendesk-forwarded message naming a player ID, a "Betler campaign name/Bonus name" field (often free text, not a real campaign name — see `references/campaign-resolution.md`), and an issue description. Skip it if it already has a `done-1` reaction or a substantive human reply in-thread — someone's already handled it.

2. **Resolve the actual campaign/bonus identity.** The ticket's name field is rarely a literal, lookup-able campaign_id. Read `references/campaign-resolution.md` for how to turn a campaign name, promo code, XP flow name, or "KYC offer"/"phone verification"/"Basketaki" style label into the real Betler campaign_id (or into "this isn't a Betler campaign at all, here's who owns it").

3. **Before recommending ANY award, check whether it was already credited.** This is the single most repeated mistake in this workflow: matching a bet/action against a campaign's qualification rule only proves it *should* qualify — it says nothing about whether the system already acted on that. Pull both `customer_campaign_status` (for `status: qualified`) and `bondet_state` (filtered by the campaign_id/bonus_id) for that player before writing "recommend award." If a credit already exists, the answer is "already credited on [date], check your account," not an award.

4. **Investigate thoroughly — don't stop at "inconclusive."** If there's a tool call you haven't made that would resolve the question (a `messages_scan` window not yet scanned, a campaign lookup not yet done, a specific bet not yet traced), make it. Genuinely say "inconclusive" only when no tool can retrieve the data at all (an image you can't view, a campaign that doesn't exist anywhere after an exhaustive search). See `references/investigation-method.md` for the query techniques that make this tractable (and the tool gotchas that silently produce false "nothing exists" answers if you're not careful).

5. **Check against the known-bug catalog before treating anything as novel.** A large fraction of tickets are recurring, already-diagnosed patterns — settlement timing at a window's close, campaign contention between two offers, a display-rounding bug, a missing event in a whitelist. Read `references/known-bugs-and-patterns.md` and match the ticket's symptoms before concluding you've found something new. Matching a known pattern doesn't mean skip verification — it means you already know *what* to verify.

6. **Apply the standing policies.** Bonus-swap requests, expired-unused bonuses, late qualification, zero-enrollment tickets, and a few other recurring situations have settled default answers — see `references/policies-and-tagging.md`.

7. **Reply in-thread, in the standing format**, and tag the right team. See "Reply format" below and `references/policies-and-tagging.md` for the full tagging decision table with exact Slack IDs.

## Reply format

Every reply follows this shape:

```
Hi <FirstName>,

<Findings from the investigation — specific IDs, timestamps, amounts, what you checked.>

<Explicit recommendation: award X, no award because Y, or (for a genuine blocker) what's needed to proceed. Never leave this implicit — don't present findings and stop short of a verdict when the evidence supports one.>

<Tag(s), if any — see the tagging table. Goes at the END, never the opening line.>
```

- **Address the CS agent who filed the ticket by first name** (from the Zendesk "X in Ticket #N" line) — not "@crm-ops-greece checked #N...". The tag is not the audience for the opening sentence.
- **Keep it short** (team feedback, approved format): 1-2 lines of findings with only the IDs, amounts and Greek times that matter, then the verdict in **bold** on its own line, then the tag. No "I checked the player in Betler…" lead-in and no detail that doesn't change the answer.
- **Always give an explicit recommendation** when the investigation supports one. "Here's what I found" without a verdict is an incomplete reply.
- **VIP goodwill rule:** for VIP customers who narrowly missed an opt-in/qualification window (minutes, not hours/days), the standing policy is to generally recommend awarding as goodwill even when the system was technically correct to withhold it automatically — don't stop at "timing miss, not a bug" for a VIP without adding the goodwill recommendation, unless there's a clear reason not to (repeated abuse pattern, missed by a lot more than a few minutes).
- **If the ticket has no real issue description** (just a campaign name + ticket code, no actual question or complaint), don't infer what's being asked and answer a guess — reply asking the CS agent what specifically needs checking.
- Don't manually sign off with "Sent using Claude" — the Slack integration auto-appends that.

## Tagging, in brief

Full decision table and exact Slack IDs are in `references/policies-and-tagging.md`. The short version:

| Situation | Tag |
|---|---|
| Recommending an award, or flagging something ops needs to check/do, or surfacing a campaign problem pattern for the first time | `@crm-ops-greece` |
| Anything where the claim is Supercoins (missions paying Supercoins, coin-shop purchases, a coins prize from SuperSpin). A SuperSpin claim for regular FS/FC/cash goes to crm-ops-greece | `@loyalty-crm-ops` **instead of** crm-ops-greece |
| Serial Winner / Super Streak (same product, internal vs. external name) — fixtures, display errors, contested streaks | `@hunch-cs` (`<!subteam^S0CCR3M59RR>`), **stacked on top of** whichever of the above applies |
| Basketaki MVP, or any physical-item/loyalty-shop-merchandise fulfillment question | George Papaspyros (person mention), not a subteam |
| Plain no-award with nothing for anyone to action | **No tag at all** — don't default to crm-ops-greece "just in case" |

The Supercoins carve-out is the single most-repeated correction in this whole workflow — a mission can look like an ordinary sports ticket and still pay out in Supercoins underneath. Check the actual reward currency, not the Zendesk Topic category.

## Reference files

| File | Open this when... |
|---|---|
| `references/campaign-resolution.md` | You need to turn a ticket's free-text campaign name, promo code, XP flow name, or category label ("KYC offer", "phone verification", "Basketaki", "casino app offer") into an actual Betler campaign_id — or determine it isn't a Betler campaign at all. |
| `references/investigation-method.md` | You're about to query Betler/OpenSearch and want to avoid the tool-level gotchas that produce confident-looking false negatives (keyed lookups, partial fetches, date-boundary filters, live-vs-draft config). |
| `references/known-bugs-and-patterns.md` | The ticket's symptoms feel familiar — settlement timing, two campaigns fighting over the same event, a stuck "100%" display, a bet that "should" count but doesn't. Check here before treating it as novel. |
| `references/policies-and-tagging.md` | You've got a verdict and need to know the standing policy for swaps/re-grants/late-qualification/zero-enrollment, the exact reply format details, or which Slack ID to tag. |

Read only what the current ticket needs — most tickets only touch one or two of these files, not all four.
