# Claude Code procedure

Use this procedure when the host provides Claude Code's Agent and SendMessage tools plus background-agent completion
notifications.

## Presets and seat agents

| Preset | Voting members | Tenth man | Runs per round |
|---|---|---|---|
| `light` | `fable`, `opus`, `sonnet` | strongest member (`fable` as of October 2026) | 4 |
| `standard` (default) | `fable`, `opus`, `sonnet`, `opus-5`, `opus-4-8` | strongest member (`fable` as of October 2026) | 6 |
| `no-fable` | `opus`, `sonnet`, `opus-5` | strongest member (`opus`) | 4 |

The plugin ships role-neutral agents named `tenth-man:seat-fable`, `tenth-man:seat-opus`,
`tenth-man:seat-sonnet`, `tenth-man:seat-opus-5`, and `tenth-man:seat-opus-4-8`. When installed by hand, omit the
`tenth-man:` prefix. Their corresponding models are `fable`, `opus`, `sonnet`, `claude-opus-5`, and
`claude-opus-4-8`.

Model availability and rankings change. If a pinned model is unavailable, drop the seat and tell the user; do not
silently substitute. If an added model's rank is unknown and affects the dissenter assignment, ask at plan approval.

## Start, collect, and continue

- Start every seat in the background with Agent. Give it its role, the brief and material paths in read order, and a
  unique reply path. Record every agent ID because one seat agent runs twice when it supplies both a member and the
  tenth man.
- Seat agents write their complete replies to their assigned paths. They must not edit any other file.
- On each completion notification, verify that the seat's reply file exists and mark only that seat complete. Do not
  tally, summarize, or disclose partial results until every expected reply file exists.
- After every valid reply file exists, tally the round. Continue the same seats with SendMessage by agent ID, retaining
  role and model assignment.
- If context has become excessive or the council goes beyond three rounds, start fresh seats and pass earlier briefs
  and preserved replies as files. Record replacements in the minutes.

Track a 30-minute deadline for each round. If it expires, identify missing seats, inspect their status, and retry once
with the same inputs. A seat that completed without writing its response file is still missing.
