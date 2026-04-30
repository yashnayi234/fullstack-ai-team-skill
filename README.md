# Full-Stack AI Team — A Claude Skill

> Turn Claude into a coordinated 7-person product team: PM, System Designer, Research Engineer, AI Engineer, ML Engineer, Backend Engineer, and Frontend Engineer — working together on your product.

[![Skill](https://img.shields.io/badge/Claude-Skill-D97757)](https://www.anthropic.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## Why this skill exists

Most people using Claude for product work hit the same wall: Claude defaults to being a single helpful generalist. Ask it to "build me a dashboard" and you get one voice answering — usually skipping the product scoping, half-glossing the architecture, and jumping straight to code that may or may not match what you actually need.

Real product work doesn't happen that way. A real team has a PM who pushes back on scope, a system designer who picks the stack, engineers who own their layer, and the discipline to hand off cleanly between roles. This skill makes Claude operate that way.

When you install it, Claude responds to product requests as a coordinated team:

- **[PM]** scopes the work and defines success before anyone codes.
- **[SD]** picks the stack, sketches the architecture, and flags trade-offs.
- **[AIE]** / **[MLE]** owns AI and ML pieces — prompts, RAG, training, serving.
- **[BE]** / **[FE]** ship the actual code, building to the design contract.
- **[RE]** activates when the problem is novel or research-heavy.

Each role tags its section so you know who's "talking," but you get one cohesive response — not seven chatbots in a trenchcoat.

## Who this is for

- **Solo founders and indie hackers** who need product/architecture rigor without a real team.
- **Engineers building side projects** who want a PM voice pushing back on scope creep.
- **AI/ML engineers** working across the stack and tired of context-switching tones.
- **Anyone using Claude to ship real software** who wants opinionated, role-driven output instead of generic answers.

## What it changes

| Without the skill | With the skill |
|---|---|
| "Here are five options, which do you prefer?" | "[SD] Use Postgres + FastAPI. Here's why, here's the schema." |
| Jumps straight to code, skips the why | [PM] scopes first, then [SD] designs, then engineers build to that design |
| One voice, one register | Role-tagged sections so you can see *who* owns each decision |
| Risks buried in prose | Explicit `[Role] — Risk:` callouts |
| Forgets product context between turns | Roles reference each other's prior output (FE builds to BE's API contract) |

## Installation

### Option A — Claude.ai (simplest)

1. Copy the contents of [`skill/SKILL.md`](skill/SKILL.md).
2. In Claude.ai, open **Settings → Capabilities → Skills**.
3. Create a new skill, paste the contents, and save.

### Option B — Claude Code

```bash
git clone https://github.com/yashnayi234/fullstack-ai-team-skill.git
cp -r fullstack-ai-team-skill/skill ~/.claude/skills/fullstack-ai-team
```

Restart Claude Code and the skill will be auto-discovered.

### Option C — Project-scoped (Claude Code projects)

Drop the `skill/` directory into your project's `.claude/skills/fullstack-ai-team/` folder. The skill activates only inside that project.

## Quick start

Once installed, just describe what you want to build. The skill triggers automatically on product, architecture, and build-related requests:

> "Help me build a tool that summarizes research papers."

> "Design an architecture for a real-time chat app that scales to 10k concurrent users."

> "I have a CSV of customer support tickets — build me a triage system using LLMs."

> "Should I use a vector DB or full-text search for this?"

You don't need to explicitly say "act as a team" — the skill description handles triggering. If you want to force a single role, just ask for it: "Just the BE perspective: design the API."

See [`examples/`](examples/) for full input/output walkthroughs.

## How it works under the hood

Skills in Claude work via [progressive disclosure](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview):

1. The skill's **description** lives in Claude's context at all times (~100 words). When your message matches the description, Claude pulls in the full skill.
2. The **SKILL.md body** loads, telling Claude how to operate as the team — the roles, the rules, the response format.
3. Claude responds in-character as the team for that turn and subsequent turns in the conversation.

The skill is intentionally written to be **opinionated** — picking React/Next.js as the default frontend, Postgres as the default DB, etc. — because real teams don't dither. You can override any default in your prompt.

## Customizing

The skill is one markdown file. Fork the repo and tweak it:

- **Change defaults**: Don't want React? Edit the `[FE]` section in `SKILL.md`.
- **Add roles**: Need a `[Designer]` or `[DevOps]` role? Add them with the same pattern.
- **Tighten triage**: If you find Claude over-ceremoning small tasks, strengthen the "skip the ceremony" guidance.
- **Industry-specific**: Add domain context (healthcare, fintech, gamedev) to make the team's defaults match your space.

## Examples

The [`examples/`](examples/) directory contains three end-to-end walkthroughs:

1. **[Feature build](examples/example-1-feature-build.md)** — Going from "I want to build X" to scoped requirements + working code.
2. **[Debug session](examples/example-2-debug-session.md)** — How the team handles a single-role task without ceremony.
3. **[Architecture review](examples/example-3-architecture-review.md)** — SD-led review of an existing system with risk callouts.

## Contributing

Contributions welcome. See [CONTRIBUTING.md](CONTRIBUTING.md). The most useful PRs are:

- New examples showing the skill in action on different problem types.
- Tweaks to role definitions that make outputs sharper.
- Bug reports with concrete prompts where the skill underperforms.

## License

[MIT](LICENSE) — use it, fork it, ship it.

## Credits

Built by [Yash Nayi](https://github.com/yashnayi234). Inspired by the pattern of treating LLMs as multi-role teams rather than single generalists, and shaped through real use building [Auron](https://github.com/yashnayi234) and other products.

If this skill helps you ship something, I'd love to hear about it — open an issue or tag me.
