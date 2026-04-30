# Example 2 — Debug session: "My useEffect runs twice"

A small, single-role task. The skill should *skip* the team ceremony and just answer. Showing this is important because over-ceremonied responses are the #1 way multi-role skills go wrong.

---

## Prompt

> My `useEffect` is firing twice on mount in dev. Is this a real bug?

## Response

**[FE]** — That's React 18 Strict Mode in development, not a bug. Strict Mode intentionally double-invokes effects to surface effects that aren't cleanup-safe. Three options:

1. **Leave it alone.** It only happens in dev, and it's catching a real category of bugs. This is the recommended default.
2. **Make your effect idempotent or add proper cleanup**, which is what Strict Mode is nudging you toward:
   ```tsx
   useEffect(() => {
     const ctrl = new AbortController();
     fetchData({ signal: ctrl.signal });
     return () => ctrl.abort();
   }, []);
   ```
3. **Disable Strict Mode** by removing `<React.StrictMode>` from your root. Don't. You'll regret it later.

If the double-fire is causing visible bugs (duplicate API writes, doubled analytics events), the fix is option 2, not option 3.

---

## What the skill did well here

- **No PM scoping.** This isn't a product question.
- **No SD architecture section.** No system to design.
- **One role, opinionated answer.** "Don't disable Strict Mode" — picked a side instead of listing options neutrally.
- **Code only where it adds value.** The cleanup pattern, not boilerplate.

This is the failure mode to watch for: if a skill ceremoniously activates `[PM]` and `[SD]` for "my useEffect runs twice," the skill is over-triggering and needs tuning.
