---
name: efficiency-router
description: Use for any multi-stage task (video, code, writing, research, analysis, data work). Splits the job into stages, routes each stage to the cheapest model that can do it (Haiku/Sonnet/Opus, with Fable as last resort, + effort), verifies the result, escalates on failure, and logs the outcome. Trigger when starting or continuing a job with several steps, when cost or speed matters, or when the user says "route", "router", "goat" or "be efficient".
---

# Efficiency router

Never run a whole job on one model. Split it into stages, route each stage, verify, escalate.
Skip routing for a one-step task: just do it.

## 1. Classify each stage (ask in order, first yes wins)

1. Does it decide strategy, design, story, voice, look, architecture, or anything where taste or judgment drives the result? -> **Opus 5.5, effort high**
2. Has this stage already failed once on a cheaper model? -> **Opus 5.5, effort high**
3. Is it a lookup, list, rename, probe or quick check (find a file, read a value, measure)? -> **Haiku 4.5, effort low**
4. Otherwise it is mechanical (build, render, implement from a clear spec, format, transform, summarize, draft from an outline, follow a proven template) -> **Sonnet 5.5, effort medium**

Examples of judgment stages: script/hook, art direction, system design, research question framing, final recommendation, tricky debugging.

## 2. Run

Run the stage with the Agent tool: `model` = haiku / sonnet / opus, `effort` = the level above. Brief it fully (goal, files, done-when); it starts cold.

## 3. Verify (the stage is not done until its check passes)

Pick a concrete, checkable test before running. Defaults by kind of work:

| Work | Check |
|---|---|
| Video script / hook | Passes the storytelling and script-formula checks; hook shows the premise in the first second |
| Video build / render | Renders without error; duration and 9:16 1080p match defaults; no empty shot openings |
| Captions | Max 6 words, none span a sentence |
| Video sound | Voice audible over music, no clipping, length matches |
| Visual art | art-assessment score at or above the last accepted result |
| Code | Builds, tests pass, the changed behavior is exercised |
| Writing | Meets the brief (length, tone, required points), no invented facts |
| Research | Every claim has a source; sources opened, not just searched |
| Data / analysis | Numbers re-computed or spot-checked against the source |
| Lookup / probe | Result names a real path or value, not a guess |

No obvious check? Define one in the brief ("done when X") before running, so verification is never skipped.

## 4. Escalate

Check fails -> rerun the same stage one tier up (Haiku -> Sonnet -> Opus -> Fable 5.2, effort high) with the failure message in the brief. Fable is the last tier: it is never a first choice. Fable fails -> stop and tell the user what failed. Never retry the same tier twice.

## 5. Log

Append one line per stage to `~/.claude/efficiency-router-log.md`:
`date | job | stage | model/effort | pass/fail | escalated to`

After ~10 jobs, read the log. A stage kind that escalates more than half the time moves up a tier by default. One that never escalates may move down.

## 6. Report

End each stage with one line: model used, check result, escalation if any.
