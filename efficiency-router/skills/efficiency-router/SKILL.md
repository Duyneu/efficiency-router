---
name: efficiency-router
description: Use for every What If video job (and any multi-stage job worth routing). Routes each stage to the cheapest model that can do it (Haiku/Sonnet/Opus + effort), verifies the result, escalates on failure, and logs the outcome. Trigger when starting, continuing or fixing a What If video, or when the user says "route", "router" or "goat".
---

# Efficiency router

Never run a whole video on one model. Split it into stages, route each stage, verify, escalate.

## 1. Classify each stage (ask in order, first yes wins)

1. Does it decide story, hook, voice or look (script, hook, art direction, first video in a new style)? -> **Opus 5.5, effort high**
2. Has this stage already failed once on a cheaper model? -> **Opus 5.5, effort high**
3. Is it a lookup, list, rename or probe (file search, ffprobe, duration check)? -> **Haiku 4.5, effort low**
4. Otherwise it is mechanical (scene build, render, captions, sound, proven format, small fix) -> **Sonnet 5.5, effort medium**

## 2. Run

Run the stage with the Agent tool: `model` = haiku / sonnet / opus, `effort` = the level above. Brief it fully; it starts cold.

## 3. Verify (the stage is not done until its check passes)

| Stage | Check |
|---|---|
| Script / hook | Passes storytelling-rules and ifology-script-formula memory checks; hook shows the premise in the first second |
| Scene build / render | File renders without error; ffprobe duration and 9:16 1080p match defaults; no empty shot openings |
| Captions | Max 6 words, none span a sentence (use --script) |
| Sound | Voice audible over music, no clipping, length matches video |
| Art | art-assessment score at or above the last accepted video |
| Lookup / probe | Result names a real path or value, not a guess |

## 4. Escalate

Check fails -> rerun the same stage one tier up (Haiku -> Sonnet -> Opus) with the failure message in the brief. Opus fails -> stop and tell the user what failed. Never retry the same tier twice.

## 5. Log

Append one line per stage to `~/.claude/efficiency-router-log.md`:
`date | video | stage | model/effort | pass/fail | escalated to`

After every ~10 videos, read the log. A stage that escalates more than half the time moves up a tier by default. A stage that never escalates may move down.

## 6. Report

End each stage with one line: model used, check result, escalation if any.
