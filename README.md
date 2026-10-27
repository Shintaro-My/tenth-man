# tenth-man

A multi-model deliberation plugin (a skill plus seat agents) for [Codex](https://developers.openai.com/codex) and
[Claude Code](https://claude.com/claude-code). Use it for a design choice, policy review, or other decision that should
not rest on one model's answer. Seats on different models vote and one fixed seat always argues against, so the council
is less likely to agree while sharing the same blind spot.

[日本語](README.ja.md)

Voting members assess every proposal. A separate **tenth man** must argue against every proposal regardless of its own
view and state what evidence would refute each objection. The main session chairs three rounds, resolves strong
concerns, and writes minutes showing what was agreed, why, and what remains open.

The skill always presents its council plan first and waits for your approval. Running a council never applies its
conclusions to your project automatically.

## How it works

1. The chair proposes the items, models, rules, cost, and output folder.
2. Round 1: members vote; the tenth man challenges every item.
3. Round 2: members reconsider strong concerns and answer the most material objections.
4. Round 3: members vote on draft minutes; the tenth man checks the result against the evidence.
5. The chair writes `.tenth-man/<date>-<slug>/minutes.md`.

Votes are **agree**, **agree with conditions**, **oppose**, and **strong concern**. Agreement requires a majority of
voting members and no remaining strong concern. The tenth man has no vote.

## Install for Codex

### As a plugin

```sh
codex plugin marketplace add Shintaro-My/tenth-man
codex plugin add tenth-man@tenth-man
```

For a local checkout, add its path instead:

```sh
codex plugin marketplace add /path/to/tenth-man
codex plugin add tenth-man@tenth-man
```

Invoke the plugin skill by typing `$tenth-man:council` followed by the question, or choose it from `/skills`:

```text
$tenth-man:council Should we move session state from Redis to signed cookies?
```

### As a standalone skill

Copy `skills/council` to a Codex skill directory:

```sh
cp -r skills/council ~/.agents/skills/
```

When installed this way, invoke it as `$council`.

For one repository, copy it to `<repo>/.agents/skills/` instead. This repository also contains role-neutral Codex seat
profiles under `.codex/agents/`; a checkout discovers them automatically. To use the profiles from every repository:

```sh
cp .codex/agents/seat-*.toml ~/.codex/agents/
```

The skill remains functional without that copy: it can start subagents with the same explicit model assignments and
continue their threads across rounds. Plugin packaging does not install custom-agent TOML files automatically.

## Install for Claude Code

### As a plugin

```text
/plugin marketplace add Shintaro-My/tenth-man
/plugin install tenth-man@tenth-man
```

To test a local checkout:

```sh
claude --plugin-dir /path/to/tenth-man
```

Invoke it with:

```text
/tenth-man:council Should we move session state from Redis to signed cookies?
```

### By hand

```sh
cp -r skills/council ~/.claude/skills/
cp agents/seat-*.md ~/.claude/agents/
```

The hand-installed command is `/council`; seat names omit the `tenth-man:` prefix.

## Model presets

The chair proposes `light` for a small question and `standard` otherwise. You can select a preset or change any seat
when approving the plan.

The tenth man runs on the same model as the strongest-reasoning member. The ranking comes from measured results, not
from version numbers; the tenth-man columns below reflect it as of October 2026. When a new model's rank is unknown,
the chair asks you. To use another model, say so at the go-ahead.

### Codex

| Preset | Voting members | Tenth man | Runs per round |
|---|---|---|---|
| `light` | GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra | GPT-6 Astra | 4 |
| `standard` | GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol | GPT-6 Astra | 5 |
| `no-astra` | GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol | GPT-6.1 Sol | 4 |

These defaults use only models exposed by the current Codex host and account. The chair reports unavailable models and
never silently substitutes one. If the host's concurrency limit is smaller than the council, seats run in waves from
the same unchanged brief; no tally is made until all seats finish.

The matching Codex custom agents are `seat_astra`, `seat_sol`, `seat_terra`, `seat_previous_sol`, and the optional
lower-cost `seat_luna`. Their TOML definitions pin the model and reasoning effort and keep council work read-only.

### Claude Code

| Preset | Voting members | Tenth man | Runs per round |
|---|---|---|---|
| `light` | Fable, Opus, Sonnet | Fable | 4 |
| `standard` | Fable, Opus, Sonnet, Opus 5, Opus 4.8 | Fable | 6 |
| `no-fable` | Opus, Sonnet, Opus 5 | Opus | 4 |

Every preset keeps an Opus seat even while Fable ranks higher, because Opus sometimes finds problems that Fable misses.

The Claude plugin includes five role-neutral seat agents in `agents/`. A brief assigns each run its member or tenth-man
role.

### Adding a Claude model

An older Claude model can be selected through the `model:` field of an agent definition. Copy a seat file, change
`name`, `description`, and `model`, then save it under `~/.claude/agents/` for all projects or `.claude/agents/` for one
project:

```markdown
---
name: seat-opus-4-7
description: A seat in a tenth-man council, on Opus 4.7.
model: claude-opus-4-7
effort: high
tools: Read, Grep, Glob, Bash, Write
---
```

Keep the same role-neutral body as the shipped seats. The chair can then assign `seat-opus-4-7` as a member or the
tenth man.

### Using your own Claude agents

Any existing Claude agent definition can hold a seat. The chair starts its prompt by saying that this task is a council
seat rather than the agent's usual job, then assigns the role and common brief. This is useful for review agents pinned
to an older model.

### When a pinned Claude model is retired

If a pinned model is no longer served, the seat fails to start. The chair drops it and tells you; it never substitutes a
different model silently. Delete or update the seat file to match the models available to your account.

## Cost and requirements

Each seat is one full subagent run per round. Over three rounds, the Codex `standard` preset uses 15 runs and the Claude
Code `standard` preset uses 18.

- Codex: a current release with subagents enabled and access to the selected models.
- Claude Code: Agent and SendMessage tools, background-agent completion notifications, and access to the selected
  models.

ChatGPT chat is not currently a supported target because it cannot run subagents on a different model for each seat.

## Why “tenth man”?

The skill is inspired by “the tenth man,” a fictional Israeli rule in the 2013 film *World War Z*. When nine people
reach the same conclusion from the same information, the tenth assumes that they are wrong and investigates the
opposite. The rule is fictional, but similar practices exist:

- After Israeli military intelligence misread warning signs before the 1973 Yom Kippur War, it established a function
  for challenging its own assessments, commonly called *ipcha mistabra*—Aramaic for “the opposite is likely.”
- The Talmud (Sanhedrin 17a) records that if every judge votes to convict in a capital case, the accused goes free; a
  unanimous result suggests that no one properly argued the defense.
- In Catholic canonization cases, the “devil's advocate” argued against the candidate and tested weak points in the
  claimed life and miracles. The office was formalized in 1587 and greatly reduced in 1983.

## License

MIT. See [LICENSE](LICENSE).
