# Efficiency Router

A Claude Code plugin that stops you running a whole job on one expensive model.

It splits any multi-stage job into stages, runs each stage on the **cheapest model that can do it**, **checks the result**, and **moves up a model only when a check fails**. Every stage is logged so the routing improves over time.

> Status: early version (v0.2.0). Built and used for animated "What If" videos, written to work on any multi-stage job.

## Why

Running everything on the strongest model wastes tokens and time. Most stages (building, formatting, summarizing, lookups) don't need it. A few (strategy, story, design, hard debugging) do. The router decides per stage.

## How it works

For every stage of a job:

1. **Classify**, first match wins:
   | Stage kind | Model | Effort |
   |---|---|---|
   | Judgment: strategy, design, story, taste, architecture, or a stage that already failed once | Opus 5.5 | high |
   | Mechanical: build, implement from a clear spec, format, transform, summarize, follow a template | Sonnet 5.5 | medium |
   | Trivial: lookup, list, rename, quick check | Haiku 4.5 | low |
2. **Run** the stage as a subagent on that model.
3. **Verify** with a concrete check before moving on (code builds and tests pass, writing meets the brief, research claims have sources, numbers are spot-checked, a video renders at the right size and length, and so on). If a stage has no obvious check, one is written first.
4. **Escalate** on a failed check: Haiku, then Sonnet, then Opus. Opus fails: it stops and reports. Never retries the same model twice.
5. **Log** one line per stage to `~/.claude/efficiency-router-log.md`. After about 10 jobs, stages that often escalate move up a tier by default, and stages that never escalate may move down.

One-step tasks skip routing.

## Install

Requires [Claude Code](https://claude.com/claude-code).

```bash
claude plugin marketplace add Duyneu/efficiency-router
```

```bash
claude plugin install efficiency-router@efficiency-router
```

Restart Claude Code afterwards. (The marketplace and the plugin inside it are both named `efficiency-router`.)

> If the repo is private, only its owner can install it. It has to be public for others to use these commands.

## Use

Start any multi-stage job and the skill triggers on its own. You can also ask for it: say "route", "router", "goat" or "be efficient". Each stage ends with one line: the model used, the check result, and any escalation.

## Update

```bash
claude plugin marketplace update efficiency-router
```

```bash
claude plugin update efficiency-router@efficiency-router
```

## Repo layout

```
.claude-plugin/marketplace.json         marketplace listing
efficiency-router/
  .claude-plugin/plugin.json            plugin manifest
  skills/efficiency-router/SKILL.md     the routing rules (edit this to tune behavior)
```

## Customize

Edit `SKILL.md`: change which stages go to which model, add rows to the verification table for your own kind of work, or change the log location. Bump the version in `plugin.json`, then run the update commands above.

## Known limits

- The check table is a starting set and has not been tuned on many real jobs yet.
- It relies on Claude Code's subagent `model` and `effort` options. If your setup doesn't allow them, the routing falls back to your session's model.
- Model names (Haiku 4.5, Sonnet 5.5, Opus 5.5) are hard-coded in the skill and need editing when newer models ship.

## License

MIT. See [LICENSE](LICENSE).
