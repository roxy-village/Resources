# Village Ops Playbook

Context files for running a high-velocity startup with AI-powered operations. Built by the team at [The Village](https://thevillageapp.co) during our pre-seed sprint.

## What this is

A collection of markdown files designed to be dropped into Claude (Cowork mode, Claude Code, or any LLM workspace) as context. Each file encodes a system we actually use to run our team — daily priorities, GTM execution, brand voice, sprint coaching. They're not documentation. They're executable operating procedures.

We built these because we got tired of repeating ourselves to AI tools. Instead of re-explaining our team structure, tone, priorities, and workflows every session, we wrote it down once and let Claude run with it.

## How to use these files

1. **Pick the context file** relevant to what you're building (e.g., `daily-ops-coach-setup-guide.md` for autonomous daily team priorities).
2. **Drop it into Claude** as a project instruction, skill, or paste it into your conversation context.
3. **Customize the bracketed sections** — team roster, Slack channels, Jira project, webhook URLs, scope exclusions. Every team is different; the structure is the reusable part.
4. **Test interactively first.** Run the system manually 3-5 times and fix what's wrong before you automate it. False positives destroy trust faster than missing coverage.
5. **Iterate in the open.** When you find a failure mode, add a BAD vs GOOD example directly in the file. These files get better every time they break.

## Context files

| File | What it does |
|------|-------------|
| `daily-ops-coach-setup-guide.md` | Autonomous daily priorities coach — scans Slack, Jira, and Gmail, then posts personalized priority lists per team member |
| *(more coming)* | |

## Philosophy

These files are opinionated. A few principles that run through all of them:

- **Verify before you assert.** Every system includes cross-referencing and thread-checking rules because the #1 failure mode of AI ops tools is confidently telling someone to do something they already did.
- **Tone is a feature.** We spent real debugging cycles on passive-aggressive phrasing. If your AI coach sounds like a disappointed manager, your team will mute the channel in a week.
- **Scope exclusions matter.** When an AI agent has access to your Slack, Jira, and Gmail, it will surface things that shouldn't be in a team channel. Fundraising details, legal threads, personal tasks — define what's out of bounds explicitly, with examples.
- **Skill + Scheduled Task architecture.** The skill is your workshop (easy to edit, test interactively). The scheduled task is your factory (runs autonomously on cron). Build in the workshop, ship from the factory, keep them in sync.

## Requirements

- [Claude desktop app](https://claude.ai/download) with Cowork mode, or Claude Code
- Slack MCP, Atlassian/Jira MCP, and Gmail MCP connected
- A Slack Incoming Webhook for bot-identity posting
- A team that's willing to be coached by an AI (the hard part)

## Contributing

If you adapt one of these for your team and find a new failure mode or a better pattern, open a PR. The whole point is that these get sharper with use.

## License

MIT — use it, fork it, make it yours.

---

*Built during The Village's pre-seed sprint, April 2026. We're building an app that helps people find their most compatible people and form real-world friendships. If you're curious: [thevillageapp.co](https://thevillageapp.co)*
