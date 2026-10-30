# Codex procedure

Use this procedure when the host provides Codex subagent collaboration tools. The repository ships role-neutral custom
agent profiles under `.codex/agents/`. The skill can also set the same models directly, so plugin installation does not
depend on copying those profiles.

## Presets

Use only models exposed by the current Codex host and account. These are the preferred presets as of October 2026:

| Preset | Voting members | Tenth man | Runs per round |
|---|---|---|---|
| `light` | `gpt-6-astra`, `gpt-6.1-sol`, `gpt-6.1-terra` | `gpt-6-astra` | 4 |
| `standard` (default) | `gpt-6-astra`, `gpt-6.1-sol`, `gpt-6.1-terra`, `gpt-6-sol` | `gpt-6-astra` | 5 |
| `no-astra` | `gpt-6.1-sol`, `gpt-6.1-terra`, `gpt-6-sol` | `gpt-6.1-sol` | 4 |

`gpt-6-astra` is the strongest-reasoning model in these presets. `gpt-6.1-sol` offers near-Astra performance on complex
work, `gpt-6.1-terra` adds a fast perspective tuned for visual reasoning, computer use, scoped fixes, and clear writing,
and `gpt-6-sol` retains a previous generation. This makes `gpt-6.1-sol` the default tenth man when Astra is excluded.
Do not use `gpt-6-luna` as a voting member by default because it is intended for focused, high-volume work, but allow
the user to choose it.

## Seat profiles

| Custom agent | Model | Preset use |
|---|---|---|
| `seat_astra` | `gpt-6-astra` | member and default tenth man |
| `seat_sol` | `gpt-6.1-sol` | member; tenth man in `no-astra` |
| `seat_terra` | `gpt-6.1-terra` | member |
| `seat_previous_sol` | `gpt-6-sol` | member for generation diversity |
| `seat_luna` | `gpt-6-luna` | optional lower-cost seat, not in a default preset |

The profiles define the model, high reasoning effort, read-only council behavior, and disabled apps/plugins/memories.
Codex discovers them automatically in a repository checkout. To make them personal agents, copy the TOML files to
`~/.codex/agents/`.

Model catalogs and account access change. Before presenting the plan, use the models the current host advertises. If a
preset model is unavailable, report that in the plan and propose a reduced preset (in fast mode, follow the fast-mode rule in `SKILL.md`); do not invent an alias or silently
substitute another model.

## Start a round

For each seat, spawn one subagent with:

- a unique lowercase task name such as `astra_member`, `sol_member`, or `tenth_man`;
- its matching custom agent profile when the current client exposes named profiles; otherwise use the exact approved
  model override and a strong supported reasoning effort;
- no inherited parent turns (`fork_turns: none`) when setting a direct model override; the brief and material files
  supply the complete shared context;
- the seat's role, the brief path, the ordered material paths, and a request to return the complete response in its
  final message;
- instructions not to edit files, contact other seats, invoke this skill, or spawn more agents.

The prompt should say that this is a council seat rather than the agent's normal job. Do not include any other seat's
response or a partial tally.

Codex may cap concurrent subagent threads below the number of seats. Start as many independent seats as capacity
allows, wait for completed seats, then start the remainder with the same unchanged brief. Running a round in waves does
not permit an early tally or changes to later prompts.

## Collect and continue

- Wait for mailbox updates with the longest practical wait interval. Use the agent-status view to distinguish running,
  completed, and failed seats; do not poll the filesystem with a shell loop.
- After all seats finish, the chair writes each returned final response verbatim to its canonical round file. Seats do
  not write council files themselves, which avoids concurrent edits.
- Keep the mapping from seat name to agent/thread ID. For rounds 2 and 3, send a follow-up task to each same agent with
  the new brief path and response requirements.
- A completed/idle agent can receive a follow-up task. If a thread has been closed or cannot continue, spawn a
  replacement on the same model and give it the earlier briefs and that seat's preserved responses. Record the
  replacement in `minutes.md`.
- Do not end the chair's turn while required seats are still running. Continue waiting until the round is complete or
  the failure rule in `SKILL.md` has been exhausted.

Subagents inherit the parent session's sandbox, approval settings, provider, and MCP servers. A request to run the
council does not authorize the chair or seats to request broader permissions. Keep seat work read-only even when the
parent can write.
