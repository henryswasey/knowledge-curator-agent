# Knowledge Curator Agent

Config and prompts for a Claude Code agent that reads resolved Jira tickets and keeps Confluence runbooks current, automatically.

Built and documented as part of the [Substack series on AI-first program management](#).
<!-- TODO: replace (#) with the live post URL -->

In production it gives a senior architect back about four hours a week and keeps the runbooks trustworthy, because every page is backed by tickets that were actually resolved.

---

## What this is

A two-routine agent that turns your team's resolved support tickets into living documentation:

| Routine | Cadence | What it does |
|---|---|---|
| **Mode B** (weekly) | Automated, weekly | Scans last 7 days of tickets, matches them to existing runbooks, updates child pages, flags anomalies, posts a Slack digest |
| **Mode A** (quarterly) | Manual trigger | Clusters 180 days of tickets from scratch, produces a quarterly snapshot. Also runs the one-time bootstrap that creates your initial runbook set. |

This repo is the **config store** the agent reads on every run. There is no code to run, no dependencies to install.

---

## What lives here

| File | Purpose |
|---|---|
| `prompt-mode-b.md` | Weekly routine prompt, the main giveaway. Start here. |
| `prompt-mode-a.md` | Quarterly snapshot / bootstrap prompt. Add later once Mode B is running. |
| `references.md` | Atlassian IDs, Confluence page IDs, Slack channel ID, canonical JQL strings, and dedup rules. Fill this in with your values. |
| `templates/runbook_parent.md` | Durable parent page template. Created once per runbook cluster, human-owned. |
| `templates/runbook_child.md` | Volatile child page. Rewritten every weekly run by the agent. |
| `templates/digest.md` | Weekly digest layout, Slack post + Confluence page. |
| `templates/snapshot.md` | Quarterly Mode A snapshot page. Write-once per run. |

---

## How to use it

### Prerequisites

- Claude.ai Team or Pro account with access to Claude Code Routines
- Atlassian MCP connector (Jira + Confluence)
- Slack MCP connector
- GitHub MCP connector (so the routine can clone this repo)

### Setup steps

1. **Fork or clone this repo** and make it accessible to your Claude account.
2. **Fill in `references.md`.** Replace every `[BRACKETED_PLACEHOLDER]` with your real Atlassian IDs, page IDs, and Slack channel info. The legend at the top of the file explains each field.
3. **Create the Confluence page structure** described in `references.md` (README parent page, Lessons Log, Uncategorized Working Set, etc.).
4. **Create a Claude Code Routine** at claude.ai/code → Routines. Paste the contents of `prompt-mode-b.md` as the system prompt. Point it at this repo.
5. **Run Mode A bootstrap first** (one-time): create a second Routine with `prompt-mode-a.md`, trigger it manually with the message `PHASE 3 BOOTSTRAP`, and confirm. This creates your initial runbook set in Confluence.
6. **Schedule Mode B** weekly (e.g. Friday 8 AM). Let it run.

### Placeholders legend

Every `[PLACEHOLDER]` in the prompts and templates maps to a field in `references.md`. Fill in that file and the prompts stay portable, you never need to edit them directly.

---

## Design principles

- **Config store, not code.** The agent reads this repo at runtime. Change a file, commit to main, the next run picks it up automatically.
- **Human-owned parent pages.** The agent only writes to volatile child pages and the Working Set. Parent runbook pages are yours to edit.
- **Lessons Log as memory.** A Confluence page the agent reads first on every run. Add lessons there to override defaults without touching the prompt.
- **Untrusted input by default.** Ticket text is treated as data to analyze, never as instructions to follow.
- **Halt-first on anomalies.** The agent stops and DMs the operator rather than writing bad data to Confluence.

---

## Safety and limitations

- **Durable vs volatile split.** Humans own the narrative on parent pages; the agent only maintains the evidence (counts, dates, examples, open questions) on child pages.
- **Least privilege (target state).** The prototype runs under personal credentials. Production should use a scoped service account limited to the projects, Confluence space, and Slack channel it needs.
- **Clustering is good, not perfect.** Edge cases still need human confirmation, by design.
- **A resolved ticket can lie.** Some close on a workaround, an SLA timer, or a wrong root cause, so a little noise can reach the evidence before review catches it.

---

## Adding Mode A later

Mode A is optional. Mode B runs independently. When you're ready to add quarterly snapshots, create a second Routine with `prompt-mode-a.md`. Trigger it manually once a quarter.

---

## Questions or adaptations?

This prompt is designed to be adapted. Swap Jira for Linear, Confluence for Notion, or Slack for Teams by updating the tool calls and placeholder names. The structure stays the same.
