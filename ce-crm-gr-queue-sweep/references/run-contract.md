# Run contract: constants, tools, report shape

## Constants

| Thing | Value |
|---|---|
| Channel #ce-crm-gr | `C0ABBF12G93` |
| Ticket parent author | Zendesk bot, user `U04KFLLEUSD` (header: `<Name> in Ticket #NNNNNN`) |
| Message identifier | `Message TS` on the parent (use as `thread_ts` and as `message_ts` for reactions) |
| Reaction: picked up | `eyes` |
| Reaction: done (closes the task) | `done-1` — set by humans, AND by the sweep itself (since 2026-11-12) when its reply is the final answer (no-award, already credited, "wait 24h and reopen"); never when it recommends an award, tags a team to act, or still needs something from the requester/team |
| Tag: CRM OPS Greece | `<!subteam^S0AAXR6SJKU>` |
| Tag: Loyalty CRM OPS (Supercoins only) | `<!subteam^S0BGAAABDPU>` |
| Tag: Hunch CS (all Hunch topics: Serial Winner / Super Streak fixtures, settlement, streak disputes, display) | `<!subteam^S0CCR3M59RR>` (user group "hunch-cs-bot"; replaces old @hunch-team `S06R99UPZS8` as of 2026-11-12) |
| Tag: Casino tournament owners | `<!subteam^S0BJUUP3RQT>` |
| Person: physical prizes / Basketaki | `<@U0AH4BR70P6>` (George Papaspyros) |

Copy the tag IDs from this table every time, never from memory (a wrong ID renders as an unknown group and needs a follow-up message). Subteam IDs start with `S` and MUST use the `<!subteam^ID>` form. `<@ID>` renders as plain text.

## Tools

- Slack: `slack_read_channel`, `slack_read_thread`, `slack_add_reaction`, `slack_send_message` (param `message`, `thread_ts`), `slack_read_file` (screenshots).
- Betler production MCP: `opensearch_search` (default for "does this bonus/event exist"), `lookup` (`topic=campaign_definitions`, param `kafka_key`; big results are saved to a file, parse with python), `opensearch_indices`.
- Snowflake read-only SQL: campaign name → id via `PROD.DWH.D_CAMPAIGN ... ILIKE`.

Useful OpenSearch indices: `customer_attributes`, `customer_campaign_status-*`, `customer_campaign_tick-*` (per-bet qualified/eligible events), `noncritical_honey_tickets_history-*` (SuperSpin tickets), `bondet_state`, `gaming_ticker-YYYY.MM.DD`, `payment_with_country-*`, `transactions_with_country-2026.10.*` (term `metadata.ticket_code.keyword`). ID fields need `.keyword`.

## Ordering and idempotency

- Process oldest open ticket first.
- Idempotency key = (Message TS of the parent). Before posting, confirm no reply from the Claude-authored account already exists in that thread unless a later human message asks a new question.
- The 👀 reaction is not a lock. Other agents or people may also add it; it only means "seen".

## Run report (JSON)

```json
{
  "run_at": "2026-10-22T11:30:00+03:00",
  "mode": "post",
  "blocked": null,
  "tickets": [
    {
      "ticket": 556950,
      "ts": "1792646651.117589",
      "initiator": "Kostis",
      "player_id": "e5f55a3f-...",
      "verdict": "no_award",
      "tag": null,
      "action": "posted",
      "reason": "Acca Boost already inside the €442.71 gross payout; tax €13.90 deducted",
      "evidence": ["transactions_with_country 2026.10.22 settle|...", "ticket 8933-S1ADUZ"]
    }
  ],
  "skipped": [{"ticket": 554978, "why": "done-1"}],
  "deferred": [],
  "learned": ["…new pattern worth saving, or empty"]
}
```

`verdict`: `award` | `no_award` | `already_credited` | `needs_info` | `escalate`. `action`: `posted` | `drafted` | `skipped_answered_meanwhile` | `blocked`. `blocked` is `teleport_expired` | `slack_unreachable` | null.

## Human-readable summary (end of every run)

```
Sweep 11:30 (post mode): 3 new, 2 replied, 1 deferred
- #556950 Kostis — no award (Acca Boost already in payout)
- #556918 Stavroula — no award (needs ONE €50+ deposit; €70 split across 5 deposits doesn't count)
- #556919 George — AWARD 40 FS, tagged crm-ops-greece
Skipped: #554978 (done-1)
```

Put anything blocked first, e.g. `BLOCKED: Betler session expired — run tsh login, then re-run`.
