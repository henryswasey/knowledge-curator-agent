# references.md

Operational reference for the [YOUR_TEAM_KNOWLEDGE_CURATOR] routine. The routine reads this file on every run. Keep entries terse and machine-readable where possible.

**Last updated:** 2026-06-10
**Maintained by:** [OPERATOR]

---

## Atlassian context

| Field | Value |
| --- | --- |
| Cloud ID | `[CLOUD_ID]` |
| Site | https://[SITE].atlassian.net |
| Jira projects (sources) | `[EXAMPLE_PROJECT_2]` ([EXAMPLE_PROJECT_2_NAME] — primary), `[EXAMPLE_PROJECT_1]` (deployment-required) |
| Confluence space (destination) | `[SPACE_KEY]` — space ID `[SPACE_ID]` |

## Confluence page IDs ([YOUR_TEAM_KNOWLEDGE_CURATOR])

| Page | ID | Role |
| --- | --- | --- |
| README — [YOUR_TEAM_KNOWLEDGE_CURATOR] | `[README_PAGE_ID]` | Root page. Parent of all [YOUR_TEAM_KNOWLEDGE_CURATOR] pages. |
| [YOUR_TEAM_KNOWLEDGE_CURATOR] — Lessons Log | `[LESSONS_LOG_PAGE_ID]` | Memory mechanism. **Read FIRST on every run.** Routine never writes here. |
| Uncategorized Tickets Working Set | `[UNCATEGORIZED_PAGE_ID]` | Staging page. Rewritten wholesale each Mode B run. |
| Archived Runbooks | `[ARCHIVE_PAGE_ID]` | Stale-runbook destination after human go-ahead. |
| Quarterly Snapshots — Mode A | `[SNAPSHOTS_PAGE_ID]` | Write-once snapshot destination. Mode A only. |
| Routine Reference — [YOUR_TEAM_KNOWLEDGE_CURATOR] | `[ROUTINE_REFERENCE_PAGE_ID]` | Bus-factor safeguard. Mirrors both routine prompts. Read by humans, not the routine. |

Runbook parent/child page IDs are assigned at Phase 3 bootstrap. Routine discovers them by listing children of the [YOUR_TEAM_KNOWLEDGE_CURATOR] README and matching titles — do not hardcode.

## Slack

| Field | Value |
| --- | --- |
| Channel | `[DIGEST_CHANNEL_NAME]` |
| Channel ID | `[DIGEST_CHANNEL_ID]` |
| Workspace | https://[SITE].slack.com |

Weekly digest posts go to the channel. Halt-condition alerts go as DMs to [OPERATOR].

## Canonical JQL strings

Both must be executed verbatim. The routine MUST echo the exact JQL it ran before reporting any counts.

### Mode A — 180-day window (snapshot only after Phase 3 bootstrap)

**[EXAMPLE_PROJECT_2]:**

    project = "[EXAMPLE_PROJECT_2]" AND "Request Type" in ("[REQUEST_TYPE_1]", "[REQUEST_TYPE_2]") AND status in ("Done", "Resolved", "Closed") AND resolved >= -180d

**[EXAMPLE_PROJECT_1]:**

    project = "[EXAMPLE_PROJECT_1]" AND status in ("Done", "Resolved", "Closed") AND resolved >= -180d

### Mode B — 7-day window (weekly steady state)

**[EXAMPLE_PROJECT_2]:**

    project = "[EXAMPLE_PROJECT_2]" AND "Request Type" in ("[REQUEST_TYPE_1]", "[REQUEST_TYPE_2]") AND status in ("Done", "Resolved", "Closed") AND resolved >= -7d

**[EXAMPLE_PROJECT_1]:**

    project = "[EXAMPLE_PROJECT_1]" AND status in ("Done", "Resolved", "Closed") AND resolved >= -7d

**Date filter:** `resolved` (NOT `updated`). [LEGACY_VERSION] silently substituted `updated`. See Lessons Log entry #1.

## [EXAMPLE_PROJECT_2]→[EXAMPLE_PROJECT_1] deduplication rules

Tickets often start in [EXAMPLE_PROJECT_2] and convert to [EXAMPLE_PROJECT_1] when work exceeds ~6 hours. Same underlying problem, different ticket key.

**Detection:** Inspect [EXAMPLE_PROJECT_1] ticket changelog (Atlassian:getJiraIssue with `expand=changelog`). Scan ALL changelog items where `field == "project"`. If any item has `fromString == "[EXAMPLE_PROJECT_2_NAME]"` (project id `[EXAMPLE_PROJECT_2_ID]`), the ticket is [EXAMPLE_PROJECT_2]-originated. Use the `Key` field-change event in the same history entry to extract the originating [EXAMPLE_PROJECT_2]-#### key, and use that history's `created` timestamp as the [EXAMPLE_PROJECT_2]→non-[EXAMPLE_PROJECT_2] conversion timestamp.

**Multi-hop chains ([EXAMPLE_PROJECT_2] → [EXAMPLE_PROJECT_3] → [EXAMPLE_PROJECT_1]):** Use the project event whose `fromString == "[EXAMPLE_PROJECT_2_NAME]"` as the origin, NOT the most recent project event. The [EXAMPLE_PROJECT_2] key only appears in that hop. Example: `[EXAMPLE_PROJECT_1]-####` went `[EXAMPLE_PROJECT_2]-####` → `[EXAMPLE_PROJECT_3]-####` → `[EXAMPLE_PROJECT_1]-####`. The [EXAMPLE_PROJECT_2] origin event is the first hop; the [EXAMPLE_PROJECT_3]→[EXAMPLE_PROJECT_1] hop is irrelevant for dedup attribution.

**[EXAMPLE_PROJECT_3]-originating tickets (no [EXAMPLE_PROJECT_2] predecessor):** OUT OF SCOPE for this run. Skip them. Revisit when the [EXAMPLE_PROJECT_3_NAME] scope question is resolved with [PRIMARY_RESOLVER].

**Counting rules:**
- Count ONCE, not twice.
- Earliest creation date from [EXAMPLE_PROJECT_2].
- Most recent resolution date from [EXAMPLE_PROJECT_1].
- Combined comments in chronological order.
- Both ticket keys retained; [EXAMPLE_PROJECT_1] key as primary.
- Runbook example display: `[EXAMPLE_PROJECT_1]-#### (originally [EXAMPLE_PROJECT_2]-####)`.

## Cluster thresholds (locked)

| Rule | Value |
| --- | --- |
| Cluster minimum (including Working Set entries) | 3 tickets |
| Same-week surge signal (digest watch-list, no auto-promote) | 2 tickets |
| Working Set bucket aging-out (no 2 mates found) | 12 weeks |
| Stale-runbook archive flag (zero new tickets) | 12 consecutive weeks |

## Failure-mode handling

| Failure | Action | Surface |
| --- | --- | --- |
| Tool call fails / times out | Retry once. On second failure, skip and continue. | Run metadata |
| Count anomaly (<50% or >200% of last week) | **HALT before clustering.** | Slack DM |
| Cluster below threshold (3) after dedup | Skip cluster. Note as candidate below threshold. | Run metadata |
| Lessons Log unreachable or garbage | **HALT.** Do not proceed without memory. | Slack DM |
| Digest publish failure | Retry 3× with 1-minute backoff. | Next week's run notices the gap |

## Routine context

| Field | Value |
| --- | --- |
| Runtime | Claude Code Routines (research preview, April 2026) |
| Routine names | [YOUR_TEAM_KNOWLEDGE_CURATOR] — Mode B (scheduled, paused) · [YOUR_TEAM_KNOWLEDGE_CURATOR] — Mode A (manual trigger only) |
| Mode B schedule | Weekly, Friday 8:00 AM MT (PAUSED until Phase 3 validation clears) |
| Mode A schedule | None — manual `Run now` only |
| Account owner | [OPERATOR] (claude.ai Team plan) |
| Daily run cap | 25 / rolling 24 hours |
| Source of truth for prompts | The routines themselves in Claude Desktop; mirrored in the Routine Reference Confluence page (see ID above) |

## Known infrastructure notes

- **Atlassian MCP endpoint deprecation:** The HTTP+SSE transport at `https://mcp.atlassian.com/v1/sse` is retired after **June 30, 2026**. Streamable HTTP at `https://mcp.atlassian.com/v1/mcp` is the replacement. The routine connects via the claude.ai Atlassian connector — verify it is on the new endpoint before that date.
- **[LEGACY_VERSION] artifacts:** [LEGACY_VERSION] README is at Confluence page `[LEGACY_README_PAGE_ID]` with sibling runbook drafts. Do NOT modify until Phase 3 validation completes.

## Pointers

- **[YOUR_TEAM_KNOWLEDGE_CURATOR] Build Plan:** primary spec. Maintained outside Confluence by [OPERATOR].
- **Lessons Log (`[LESSONS_LOG_PAGE_ID]`):** routine MUST read first on every run. Active lessons override defaults.
- **Routine Reference page** in [SPACE_KEY]: see ID in the Confluence page IDs table above. Mirrors both routine prompts for bus-factor coverage.
