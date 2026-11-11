# Review procedure

Use this procedure when `review` is among the leading option words. The subject is a draft that already exists. The
council does not vote on proposals; it tests the draft against objections. The tenth man goes first, and every later
step starts from its objections.

| Round | Seats | Work |
|---|---|---|
| 1 | Tenth man | Objections to the draft |
| 2 | Members | Judge each objection, propose fixes, and optionally add objections |
| 3 | Tenth man | Rebuttals to the answers and the fixes |
| 4 | Members | Final votes |

Each seat runs twice. The rules in `SKILL.md` on approval, fast mode, roles, independence, materials, language, and
failure handling apply. An unchanged draft is a successful review: no seat has a number of objections, fixes, or
rebuttals to fill, and the chair adds no round to find more.

## Prepare

- Identify the draft from the user's text: files, a diff, or text the user gave. If it cannot be identified, ask.
- Copy the draft into `materials/` so that every seat reads the same frozen version. Add only the material a reader
  needs to judge it.
- Write `brief.md` with: what the draft is and what it is for; the requirements it must meet, quoted from the user or
  from the draft itself rather than restated; this procedure and the response formats; the fixed sentence for a missing
  specific objection, in the brief's language; and the materials in read order. Do not write items, grounds, or
  arguments for the draft.
- Say in the plan and in the minutes whether the main session wrote the draft. If it did, the chair is not neutral
  about it, and the brief must not defend it.

## Files

```text
.tenth-man/20261027-migration-plan/
|-- materials/       the frozen draft and supporting material
|-- brief.md
|-- round-1/         tenth-man.md
|-- round-2/         one response per member
|-- round-3.md       tally of answers, fixes, and added objections
|-- round-3/         tenth-man.md
|-- round-4.md       draft minutes and final-vote request
|-- round-4/         one response per member
`-- minutes.md
```

## Round 1: the tenth man objects

Start only the tenth man. It returns at most six objections to the draft, most material first, each with:

- an ID (`O1`, `O2`, ...);
- the passage it targets, quoted with its location;
- the objection, stating the failure mechanism;
- the data, observation, or experiment that would refute it;
- its basis, grounded or hypothesis, as defined in `SKILL.md`.

The aiming rules under "Round 0" in `SKILL.md` apply, read for a draft instead of an item: give the objections most
specific to this draft, aim at what the draft depends on most, and do not offer a general doubt by itself. If the tenth
man finds no objection specific to the draft, it returns one objection with the fixed sentence and the basis
hypothesis. One of the objections may be a framing objection, against the draft's purpose or approach as a whole; the
tenth man marks it as such.

If the tenth man fails twice in this round, stop and tell the user. The review cannot start without its objections.

## Round 2: members judge the objections

Start the members after the tenth man's response is preserved. Give each the brief, the materials, and
`round-1/tenth-man.md`. Members do not see one another's responses. For every objection a member returns:

- one answer: **uphold** (the draft should change), **uphold in part**, or **reject** (the draft stands on this point);
- a reason of at most two sentences, which addresses the refutation condition when the materials allow checking it;
- for uphold and uphold in part, a fix: the smallest concrete change to the draft that removes the failure mechanism.

Rejecting an objection is a normal answer. A member may also add at most two objections that the tenth man did not
raise, in the round-1 format. Adding none is the normal case.

Write `round-3.md` as a neutral tally. For each objection, give every member's answer and reason, and each distinct fix
with the seats that proposed it. Merge added objections that make the same point, give them IDs (`M1`, `M2`, ...), and
list them with the seats that raised them. Do not rank, select, or comment.

## Round 3: the tenth man answers back

Continue the same tenth man with `round-3.md`. For each of its objections it either **presses** or **rests**. A rest is
not a statement of the tenth man's own view; it means it has no specific rebuttal. A press is one rebuttal of these
kinds:

- the reasons for rejecting do not meet the refutation condition;
- a proposed fix leaves the failure mechanism in place, or removes it by creating a new problem;
- the answers all rest on the same unverified premise, which the rebuttal names together with what would check it.

The tenth man raises no new objection to the draft in this round. Member-added objections go to round 4 as they are.

## Round 4: final votes

Write `round-4.md` as draft minutes. For each objection, including member-added ones, give: the quoted passage and the
objection; the round-2 tally; the tenth man's rebuttal, if any; and the candidate fixes, each written out in full and
labelled (`O1-a`, `O1-b`, ...). Continue the same members with it. For every objection a member returns:

- one final vote: **uphold**, naming one candidate fix, or **reject**;
- one sentence of reason against the text of `round-4.md`, which answers the tenth man's rebuttal when there is one.

Do not rewrite a fix after the final vote.

## Decide and record

Decide each objection from the final votes of the members that completed round 4:

- **Upheld**: a majority voted uphold. The fix is the candidate that the most of them named. If no single candidate
  has the most votes, record the objection as upheld with the fix unresolved.
- **Rejected**: a majority voted reject.
- **No consensus**: neither has a majority.

Write `minutes.md`. Copy quoted passages and fixes from `round-4.md` word for word. Record:

- whether the main session wrote the draft, the preset and models, and every choice the chair made on its own;
- upheld objections with their fixes, in the tenth man's order, followed by member-added ones;
- rejected objections, each with the members' reason and whether the refutation condition was met, was not met, or
  cannot be checked within the council;
- objections with no consensus or an unresolved fix, as open questions for the user;
- for every objection: the final vote count, each minority vote with the seat's name and its sentence of reason, and
  the tenth man's rebuttal with how members answered it;
- an upheld framing objection as an open question for the user, not as a fix.

Report the result to the user. Do not edit the draft unless the user separately asks.
