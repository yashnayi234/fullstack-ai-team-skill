# Contributing

Thanks for considering a contribution. This skill is a single markdown file plus examples — the bar to contribute is low.

## What's most helpful

1. **Examples** — Real prompts where the skill produced great (or bad) output. Add them to `examples/` with the prompt, the response, and a short note on what worked or didn't.
2. **Role tweaks** — If a role's defaults feel off (e.g., the SD picks the wrong DB by default for your domain), open a PR with the fix and a one-line rationale.
3. **Bug reports** — Open an issue with the exact prompt, the response you got, and what you expected. Include your Claude client (Claude.ai, Claude Code, API).

## What's probably out of scope

- Adding more roles for the sake of completeness. The 7 roles cover most product work; more roles = more confused output.
- Removing the role-tagging convention. It's the core mechanic.
- Hardcoding industry-specific defaults into the main skill. Fork it for that.

## PR checklist

- [ ] `SKILL.md` still under ~500 lines.
- [ ] Description still triggers correctly (test with 2-3 prompts before/after).
- [ ] If you changed defaults, note it in the PR description.
- [ ] If you added an example, link it from the README.

## Testing changes locally

The fastest way to test:

1. Edit `skill/SKILL.md`.
2. In Claude.ai or Claude Code, paste it into your skill slot (replacing the old version).
3. Run 2-3 of the example prompts from `examples/` and eyeball the output.
4. Run 1-2 prompts that *shouldn't* trigger the skill (a knowledge question, a casual chat) and confirm it stays quiet.

For more rigorous testing, use the [skill-creator](https://github.com/anthropics/skills) eval workflow.

## Code of conduct

Be kind. Critique the skill, not the contributor.
