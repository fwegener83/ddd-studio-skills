---
name: propose-context-cut
description: Propose candidate bounded contexts from a Big Picture Event Storming board in DDD Studio, with the heuristics that support each boundary. Use when a Big Picture exists but no cut does yet, or when asked to "find the contexts", "where do we cut", "propose a context cut" or "suggest bounded contexts". Produces two or three alternative cuts with evidence and open questions, published as notes and votable proposals on the board; it never changes cards or the model.
---

# Propose a context cut

The cut into bounded contexts is the most expensive decision to undo, and
it is a decision for the people who know the domain. An agent can do the
legwork: read every card, apply the heuristics consistently, and show
where they agree and where they pull apart. This skill does that and
stops there.

The heuristics are in `heuristics.md` in this folder. Read it first.

## Rules

- **Propose, do not decide.** Show two or three alternative cuts, never
  one "answer". The humans choose. Nothing in the model changes.
- **Evidence or silence.** Every boundary you draw cites the cards it
  rests on, by their text (and id when you write back to Studio). A
  boundary that only "feels right" is not reported.
- **Heuristics, not taste.** Name the heuristic behind each boundary
  (`heuristics.md` §1 to §8). When two heuristics disagree, that is a
  finding, not something to smooth over.
- **Transcribe, do not interpret.** Quote card texts as they stand. Do not
  rename events, fix the language or merge cards while reading. If two
  cards seem to mean the same thing, that is a question for the humans.
- **The rule goes with its decider.** When a policy or rule involves
  cards on both sides of a boundary, say which side decides it and where
  the board says so. If the board does not say, it is an open question.
- **Publish, do not edit.** The results go onto the board as notes and
  proposals (see "Publishing to DDD Studio"). Cards are never moved,
  changed or deleted.

## Inputs

From DDD Studio (MCP tools `mcp__ddd-studio__*`):

- the workspace: if the user did not name one, `list_workspaces` and ask;
  if the project's `CLAUDE.md` has a "DDD Studio" section, use the
  workspace it names,
- the board: `list_sketch_boards`, then `get_sketch_board` (cards,
  sections, tags, connections, and the board's notes),
- the open notes on the board: `get_note` for the ones that look like
  conflicts or language questions.

If the project has written sources (a brief, interview notes), read them
too and quote them. The board is evidence of what people said; it is
not complete.

## Steps

1. **Read the board in time order.** List the events left to right with
   their tags (phases, sources). `get_sketch_board` does not return card
   positions: take the order from phase or sequence tags, from the
   connections, or from the order the cards come back in, and say in the
   proposal where the order is inferred. List actors, external systems,
   policies, read models and which event each one is connected to. A card
   without a connection may be placed next to its event on the canvas;
   then use only what its own text or a note says, and mark the link as
   "not on the board". List the notes with the cards they point at.
2. **Mark the signals** (heuristics §1 to §7). For each heuristic, write
   down where on the timeline it fires and which cards show it:
   pivotal events, a change of actor, a change of language, the decider
   of each rule, a change of pace or time horizon, external systems, the
   people who read each read model, clusters of open questions.
3. **Form candidate contexts.** Group cards where several signals agree.
   For each candidate write: a working name in the board's own words,
   its purpose in one sentence, the events it owns, the decisions it
   makes on its own, its words, and the actors who decide in it. Leave a
   field empty rather than invent it. An empty purpose is itself a
   question for the humans: put it in the list.
4. **Form two or three alternative cuts.** Typically: one finer, one
   coarser, and one that resolves the hardest open question differently
   (for example, which side owns a disputed rule). Never include an
   option you would reject yourself; list it under "Rejected cuts" with
   the reason instead. You may mark one cut as the recommended starting
   point for the discussion; then say what would change the
   recommendation.
5. **Test each cut** against the counter-heuristics (§8): no cut by noun,
   by department alone, by process phase or by technology; no rule split
   over two contexts; no context that only passes things on. Play one or
   two cases along the timeline and count the crossings: a crossing is an
   event, command or piece of information that has to go from one
   candidate context to another for the case to go on. Count plans made
   the day before and external systems separately.
6. **Write the proposal** (format below). Open with a summary of at most
   five lines that can be read aloud (the cuts, the hardest question),
   then the full proposal.
7. **Publish** the result as notes on the board (next section), unless
   the user asked for a report only.

## Publishing to DDD Studio

By default the results go into DDD Studio as notes, so that the people can
answer them where the model is: comment, vote on a proposal, decide. If the
user asks for a report only ("report only", "nur Bericht", "dry run"), skip
this section and stop after the report.

- **Notes and comments, nothing else.** Use only `create_note` and
  `add_comment`. Never change cards, nodes, mappings or slices. Never
  resolve a note and never vote: deciding is the people's job, and the tools
  do not let an agent vote.
- **Marker.** Every note body starts with the line
  `Agent · propose-context-cut · <kind>` (`<kind>` as listed under "What"). People see at once that an agent
  wrote it, and `clear-agent-notes` finds it again. Write everything after
  the marker in the language of the model.
- **Shape of a note.** After the marker: the claim in one sentence; then
  the evidence, quoted; then in one sentence why it matters. Keep it under
  about 600 characters: notes are read on a canvas, not in a report.
- **Few notes.** At most 5 notes per run. What does not make the cut stays
  in the report in the conversation.
- **Alternatives are proposals.** Put each alternative under its note with
  `add_comment` and `proposal: true`: what would change and what it costs,
  in two or three sentences. The people vote on proposals and decide.
- **No duplicates.** Before writing, `list_notes` on the same canvas. If an
  open note with this skill's marker already makes the same claim, do not
  write it again; say so in the report. If many notes of an earlier run are
  still open, ask the user whether to clear them first with
  `clear-agent-notes`.
- **Where.** On the board that was read: `create_note` with canvas `{ view: "sketch-board", boardId }` and targets of kind `card`.
- **What.** One note "Wo schneiden wir?" / "Where do we cut?" (kind `Schnitt`/`cut`) that points at the cards on both sides of each boundary of the cuts (at most ten cards), with every cut (A, B, C) as a proposal under it: its contexts, its boundaries and its strongest argument against. Then one note for each place where the heuristics disagree and for each question only the business can answer (kind `Frage`/`question`), pointing at the cards concerned; at most four.
- **Say what you published.** End the conversation report with the number
  of notes and proposals written, and where.

## Proposal format

```markdown
# Context cut proposal: <board name>

<summary, at most five lines>

Input: workspace <name>, board <name>, <n> events, <n> notes, read <date>
Reading: <how the time order was taken; links that are not on the board>

## Signals on the timeline
| Where (between which events) | Heuristic | Evidence (card texts) |
|---|---|---|

## Candidate contexts
| Context (working name) | Purpose | Owns events | Decides | Own words | Deciders |
|---|---|---|---|---|---|

## Cut A: <name> (recommended | alternative)
- Contexts: …
- Boundaries and why: <boundary> — <heuristics> — <cards>
- What it costs: crossings per case, rules near a boundary, open risks
- Strongest argument against it: …

## Cut B / Cut C: …

## Rejected cuts
- <cut> — <why it is out, with the heuristic and the cards>

## Where the heuristics disagree
- <place> — <heuristic X says …, heuristic Y says …>

## Questions for the people who know the domain
1. <a question only the business can answer, with the cards it concerns>
```

## After the humans decide

The chosen cut is modelled as bounded contexts in DDD Studio by people
(or by an agent on their explicit instruction), with each context's
ubiquitous language. Then run `challenge-context-cut` in a fresh agent
against the result: a cut proposed by an agent gets the same independent
review as one drawn by a person.
