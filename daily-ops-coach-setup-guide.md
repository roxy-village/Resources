# Daily Ops Coach — Setup Guide

**What this is:** A system that autonomously scans your team's Slack, Jira, and Gmail every morning, then posts a personalized daily priority list for each team member to a Slack channel. It runs on Claude (Cowork mode) using two components: a **skill** (the brain) and a **scheduled task** (the executor).

**What you'll need before you start:**
- Claude desktop app with Cowork mode enabled
- Slack MCP connected (for reading channels)
- Atlassian/Jira MCP connected (for sprint data)
- Gmail MCP connected (for meeting notes / Read.AI reports)
- A Slack Incoming Webhook (for posting as a bot, not your personal account)
- ~60 minutes to configure and test

---

## Architecture: Skill + Scheduled Task

This system has two pieces that work together:

**The Skill** is the strategic brain. It contains your team roster, channel list, priority logic, tone rules, and post format. When you or Claude invoke the skill, it follows these instructions to produce the daily report. You iterate on the skill as you refine what "good" looks like.

**The Scheduled Task** is the mechanical enforcer. It runs on a cron schedule (e.g., 8am weekdays), invokes the skill's logic, and guarantees the report posts even when you're not in a Cowork session. The task prompt is self-contained — it carries the critical rules so it doesn't depend on having the skill loaded.

Why both? The skill is easy to edit and test interactively. The scheduled task is what makes it autonomous. You build and refine in the skill, then sync the critical rules into the task prompt.

---

## Step 1: Set Up Your Slack Webhook

You need a Slack Incoming Webhook so the report posts as a **bot**, not your personal account.

1. Go to https://api.slack.com/apps → Create New App → From Scratch
2. Name it whatever you want (e.g., "Ops Coach", "Daily Bot", your mascot name)
3. Pick your workspace
4. Go to **Incoming Webhooks** → Activate → **Add New Webhook to Workspace**
5. Select the channel where you want reports posted (create a dedicated channel like `#ops-agent` or `#daily-priorities`)
6. Copy the webhook URL — it looks like: `https://hooks.slack.com/services/TXXXXX/BXXXXX/xxxxxxxxxx`

Save this URL. You'll paste it into both the skill and the scheduled task.

**Why a webhook instead of the Slack MCP?** The Slack MCP posts as YOUR account. A webhook posts as the bot — cleaner for the team, and they won't think you're manually writing these at 8am.

---

## Step 2: Gather Your Configuration

Before building the skill, collect this info:

### Team Roster
For each team member, you need:

| Field | Example | Where to Find |
|-------|---------|---------------|
| Name | Jane Smith | You know this |
| Role | Head of Engineering | You know this |
| Slack User ID | U09TJHMMRRQ | Slack profile → three dots → Copy Member ID |
| Emoji | (pick one per person) | Your choice — makes the report scannable |
| Focus Area | Architecture, deploys, app stability | What they should be working on |
| Jira Account ID | 712020:xxxxx | Jira profile URL or `lookupJiraAccountId` MCP tool |

### Slack Channels to Scan
List every channel the coach should read, with channel IDs. Common categories:

- **Coordination:** #general, #standup, #blockers
- **Engineering:** #releases, #alerts, #reviews, #tech-docs
- **Design:** #design-explorations, #design-releases
- **Product:** #product-docs, #qa, #feedback
- **GTM/Marketing:** #marketing, #sales, #partnerships
- **Culture** (for Team Pulse morale moments): #random, #wins, #watercooler

To get a channel ID: right-click the channel name in Slack → Copy Link → the ID is the last segment (starts with C).

### Jira Project
- Your Jira project key (e.g., `VM`, `ENG`, `PROD`)
- Your Atlassian Cloud ID (find it at: `https://your-domain.atlassian.net/_edge/tenant_info`)

### Channels to EXCLUDE
Think about what should never appear in a team-facing post. Common exclusions:
- Fundraising channels
- Finance/legal channels
- Personal or exec-only channels
- HR channels

---

## Step 3: Build the Skill

Ask Claude to create a skill. Say something like:

> "I want to create a skill called `daily-ops-coach` that posts a daily team priorities report to Slack. Here's my setup: [paste your team roster, channel list, Jira project, webhook URL, and any exclusion rules]."

The skill should contain these sections (use ours as a template — customize everything in brackets):

### Skill Structure

```
---
name: [your-skill-name]
description: >
  Daily operations coach for [your team]. Triggers on: "run the ops coach",
  "what should [name] work on", "sprint priorities", "who's blocked", etc.
---

# [Your Team] Daily Operations Coach

## Team Roster
[Table with Name, Role, Slack ID, Emoji, Focus Area, Jira Account ID]

## How to Run

### 1. Gather Context

**Time window:** Previous calendar day only (midnight to midnight in your timezone).
Do NOT include morning-of activity.

**Slack scan:** [Your channel list with IDs]

**Jira scan:** project = [KEY] AND sprint in openSprints()
- Cloud ID: [your-cloud-id]

**Gmail scan:** Meeting notes, Read.AI reports, external threads

**Thread Resolution Check (MANDATORY):**
Before assigning any priority, read thread replies on any message that looks
like a request, blocker, or open question. A top-level message may already
be resolved in the thread.

### 2. Identify Priorities (per person, top 3)
Priority hierarchy:
1. Blockers (anything blocking another person)
2. SLA violations (stale review requests)
3. Critical-path items (what unblocks your GTM or key milestone)
4. Sprint commitments
5. Proactive improvements

**Cross-reference before assigning:**
- Check Jira status (if Done/Shipped, don't assign it)
- Check release channels (if deployed, celebrate instead)
- Check thread replies (if resolved, find the NEXT action)

### 3. Identify Blind Spots
Cross-functional dependencies, stale tickets, handoffs waiting on someone

### 4. Generate Coaching Feedback
Specific, direct, action-oriented. Name the ticket/PR/thread.

### 5. Pre-Post Refresh (MANDATORY)
Re-scan release and alert channels right before composing.
Never rely on channel reads older than 15 minutes.

### 6. Format and Post
[Your format template — see below]

## Tone Rules
- Specific (name the ticket, PR, thread)
- Honest (if there's a gap, name it)
- Enabling, not scolding (coach handing the ball, not manager wagging finger)
- Action-oriented (every bullet ends with what to DO)

BAD: "The leads list has been sitting for 3 days. Clock is ticking."
GOOD: "Leads list is ready in #marketing (posted Mon). Next step: start calls."

## Scope Exclusions
[Your hard exclusions — fundraising, legal, personal, etc.]

## How to Post
Use webhook, not Slack MCP:
curl -X POST [YOUR_WEBHOOK_URL] -H 'Content-Type: application/json' -d '{"text": "..."}'

If sandbox blocks it, use Chrome javascript_tool with fetch().
```

### Post Format Template

Customize this to your team:

```
[EMOJI] *[Team Name] Priorities — [Date]*

*Today's headline:* [One sentence — single most important priority]

---

*[Emoji] [Name] — [Role]*
1. [Priority with context]
2. [Priority]
3. [Priority]
[Blind spot icon] _Blind spot: [cross-functional thing they might not know]_
[Coach icon] _Coach note: [direct feedback on execution]_

--- (repeat for each team member) ---

*Execution System Check*
- Review SLAs: [status]
- Artifact check: [any work without tickets?]
- Silent misses: [any slipped deadlines?]

---

*Sprint & GTM Synopsis*
- Sprint: [X of Y tickets done | assessment]
- GTM: [key signals]
- Small wins: [things the team should celebrate]

---

_[Your team's motto or sign-off]_
```

---

## Step 4: Test the Skill Interactively

Before scheduling anything, run the skill manually in Cowork:

> "Run the ops coach"

Review the output. Common things to fix on first runs:

- **False priorities** — told someone to do something they already did. Fix: strengthen thread resolution checks and cross-referencing.
- **Wrong tone** — too aggressive, too passive-aggressive, too vague. Fix: add BAD vs GOOD examples to tone section.
- **Missing context** — didn't check a channel where important updates happen. Fix: add the channel to the scan list.
- **Included sensitive info** — fundraising details leaked into team post. Fix: add explicit scope exclusion rules with examples.
- **Stale data** — referenced activity from 3 days ago as "today." Fix: tighten the time window rule.

Iterate on the skill until the output feels right. This usually takes 3-5 test runs.

---

## Step 5: Create the Scheduled Task

Once the skill output looks good, create a scheduled task. Say:

> "Create a scheduled task called `daily-ops-coach` that runs at 8am on weekdays in [your timezone]. It should follow the ops coach skill to scan Slack, Jira, and Gmail, then post to Slack via webhook."

Key settings:
- **Cron:** `0 8 * * 1-5` (8am Monday-Friday, local timezone)
- **The task prompt must be self-contained.** Include your team roster, channel list, webhook URL, time window rule, tone rules, and scope exclusions directly in the prompt. The task runs outside your session and may not have the skill loaded.

### Critical rules to bake into the task prompt:

1. **Time window:** Report covers previous calendar day only (midnight to midnight). Not morning-of.
2. **Thread resolution:** Always read thread replies before flagging something as open.
3. **Pre-post refresh:** Re-scan release channels right before composing.
4. **Cross-reference:** Verify every priority against Jira status + release channels + threads.
5. **Scope exclusions:** List what must never appear.
6. **Webhook posting:** Include the full curl command or fetch() for the webhook.

---

## Step 6: Verify and Iterate

After the first scheduled run:

1. **Check the post** — does it accurately reflect yesterday?
2. **Ask team members** — "Was anything wrong or missing?"
3. **Fix false positives first** — nothing kills trust faster than telling someone to do something they already did.
4. **Update both the skill AND the task prompt** when you make changes. They can drift if you only update one.

### Common failure modes and fixes:

| Problem | Root Cause | Fix |
|---------|-----------|-----|
| "Do X" when X is already done | Didn't read thread replies | Strengthen thread resolution check |
| Stale by the time it posts | Time window too broad | Bound to previous calendar day only |
| Passive-aggressive tone | No tone examples | Add explicit BAD vs GOOD examples |
| Included fundraising info | No scope exclusions | Add hard exclusion rules with examples |
| Missing a team member | Not in roster | Add them to roster in skill AND task prompt |
| Emoji/formatting wrong | Skill and task prompt out of sync | Update both when making changes |
| Deployed at 8:15 PM, told to deploy at 8 AM | Pre-post refresh missing | Add mandatory re-scan before composing |

---

## Maintenance

- **When you add/remove team members:** Update the roster in both the skill and the scheduled task prompt.
- **When you add Slack channels:** Add them to both the skill scan list and the task prompt.
- **When the tone feels off:** Add new BAD vs GOOD examples to the tone section.
- **When you change your milestone targets:** Update the goals/metrics section so priorities stay aligned.

The skill is your workshop. The scheduled task is your factory. Build in the workshop, ship from the factory, and keep them in sync.

---

## Required Integrations Checklist

Before your first run, confirm these are connected in Claude/Cowork:

- [ ] **Slack MCP** — for reading channels and threads (search for "Slack" in connectors)
- [ ] **Atlassian/Jira MCP** — for sprint board data (search for "Atlassian" or "Jira")
- [ ] **Gmail MCP** — for meeting notes and external signals (search for "Gmail")
- [ ] **Slack Incoming Webhook** — for posting as bot (set up in Step 1)
- [ ] **Chrome extension** — backup for webhook posting if sandbox blocks curl

---

*Built by Roxy Kellogg @ The Village. Questions? Ping me.*
