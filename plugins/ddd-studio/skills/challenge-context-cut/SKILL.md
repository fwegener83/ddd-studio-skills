---
name: challenge-context-cut
description: Challenge the cut into bounded contexts and the context map in DDD Studio before anything is built on it. Use at the end of strategic design, before work that builds on the cut, after a context is added, merged or split, after a mapping is turned, or when asked to "challenge", "stress-test" or "review the domain cut / context cut / context map". Produces findings with evidence and alternatives, published as notes and votable proposals on the context map; it never changes the model.
---

# Challenge the context cut

The cut is the decision that is most expensive to undo later. DDD Studio
draws it, but it does not argue with it. This skill does.

The heuristics are in `heuristics.md` in this folder. Read it first.

## Rules

- **Independent.** Run the challenge in a fresh agent (a subagent) that
  gets only the workspace, the sources and this skill, not the
  reasoning of whoever drew the cut. The author defends the cut without
  noticing. If you are the author and cannot hand it off, say so in the
  report ("Run by: author").
- **Evidence or silence.** Every finding quotes the element and the
  passage it is based on: a glossary entry, a mapping description, a
  policy rule, a card on the board, a sentence of the sources. A finding
  without a quote is not reported. "Could be a problem some day" is a
  note, not a finding. Notes in Studio are someone's open doubt, not a
  statement of the model: quote them to support a finding, never as its
  only evidence.
- **Business before technology.** When a rule crosses a boundary, the
  question "whose rule is it?" comes before "which way do the
  dependencies point?". Do not let the cheapest technical fix decide who
  owns a rule.
- **Propose, do not decide.** Change nothing in the model or in the repo.
  The findings go onto the context map as notes and proposals (see
  "Publishing to DDD Studio"); the people decide. Every serious finding comes with at least one
  alternative and what it costs. Business questions go out as questions
  for the people who decide; architecture questions as proposed
  decisions with a recommended option.
- **Say what holds.** A report that finds only faults is not believed.
  Name the parts of the cut that survive the tests and why.

## Inputs

Read the current state from DDD Studio (MCP tools `mcp__ddd-studio__*`):

- the workspace: if the user did not name one, `list_workspaces` and ask;
  if the project's `CLAUDE.md` has a "DDD Studio" section, use the
  workspace and the conventions it names,
- contexts, subdomains, aggregates, policies: `get_model_overview`, then
  `get_node` on every bounded context (its `ubiquitousLanguage`), every
  policy (`handles`, `emits`, `constraints`), every read model that
  projects events of another context (its projection rules), and every
  aggregate whose invariants mention another context's words. In DDD
  Studio policies, event handlers and process managers live under the
  domain, not under a context: attribute them by the events they handle,
  the commands they emit and the slices they appear in, and treat a rule
  whose owner the model does not name as a question,
- mappings: `list_mappings` (type, direction, description),
- the discovery boards: `list_sketch_boards`, `get_sketch_board` (events,
  rules, notes),
- the flows, if any: `list_flows`, `list_slices` (which contexts a story
  crosses, and in which order),
- the open notes on the context map and the boards.

If the project has written sources (requirements, a log of business
decisions, architecture decision records), read them and quote them. If
the model has been exported (`export_esdm`) into the repo, the export is
what a review can quote by file and line; use it when it is current.

## Steps

1. **Canvas.** For each context write a short Bounded Context Canvas
   (heuristics §1): purpose, classification, the decisions it makes,
   what it receives and sends, its words. Leave a field empty rather
   than invent it. A context whose purpose and decisions stay empty is
   already a finding.
2. **Language** (heuristics §2). Compare the glossaries with each other
   and with the sources and the board. Look for words of one context
   inside another context's definitions, one word with two meanings in
   one context, and two words for one thing.
3. **Ownership of rules** (heuristics §3). List every policy, invariant
   and threshold that involves more than one context. For each: who
   decides, who may change it, where the sources say so. A rule split
   over two contexts, or owned by nobody in the sources, is a finding.
4. **Dependencies** (heuristics §4). Build the dependency graph from the
   mappings *and* from who reacts to whose events (policies, read models
   that project another context's events). Report every cycle, even when
   every arrow is "only an event".
5. **Scenarios** (heuristics §5). Play three to five concrete cases
   through all contexts (use the flows where they exist), at least one
   with timing and one exception from the sources. Count boundary
   crossings and round trips. Then name three changes the sources make
   likely and count how many contexts each one touches.
6. **Patterns and anti-patterns** (heuristics §6). Check that each
   mapping pattern is real (a customer–supplier has a negotiation, a
   shared kernel has a shared, jointly changed part, an anti-corruption
   layer translates something) and look for the anti-patterns.
7. **Alternatives.** For each serious finding, sketch at least one other
   cut (move the rule, turn the arrow, merge, split) and what it costs.
8. **Report** (format below). Give the user the findings and the list of
   questions in the conversation, in the language of the user's request;
   the headings may be translated. Save the report as a file only where
   the project says (its `CLAUDE.md`) or the user asks; a common place is
   `docs/reviews/<label>-context-cut.md`.
9. **Publish** the findings as notes on the context map (next section),
   unless the user asked for a report only.

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
  `Agent · challenge-context-cut · <kind>` (`<kind>` as listed under "What"). People see at once that an agent
  wrote it, and `clear-agent-notes` finds it again. Write everything after
  the marker in the language of the model.
- **Shape of a note.** After the marker: the claim in one sentence; then
  the evidence, quoted; then in one sentence why it matters. Keep it under
  about 600 characters: notes are read on a canvas, not in a report.
- **Few notes.** At most 7 notes per run. What does not make the cut stays
  in the report in the conversation.
- **Alternatives are proposals.** Put each alternative under its note with
  `add_comment` and `proposal: true`: what would change and what it costs,
  in two or three sentences. The people vote on proposals and decide.
- **No duplicates.** Before writing, `list_notes` on the same canvas. If an
  open note with this skill's marker already makes the same claim, do not
  write it again; say so in the report. If many notes of an earlier run are
  still open, ask the user whether to clear them first with
  `clear-agent-notes`.
- **Where.** On the context map: `create_note` with canvas `{ view: "context-map" }` and targets of kind `node` (the bounded contexts or external systems concerned) or `mapping` (the context mapping concerned).
- **What.** One note per finding of severity blocking or should fix (kind `blockierend`/`blocking` or `sollte behoben werden`/`should fix`), at most six, with its alternatives as proposals. A question for the business that belongs to a finding goes into that note as its last line. Then one note `Was hält`/`What holds` (kind `hält`/`holds`) pointing at the parts of the cut that passed, with the tests they passed, so that the map does not only show faults.
- **Say what you published.** End the conversation report with the number
  of notes and proposals written, and where.

## Report format

```markdown
# Context cut challenge: <label>

Input: workspace <name>, read on <date> (latest saved version: <label>,
if any; say if it is unknown whether the current state equals it),
<what was read>
Run by: <fresh agent | author>, <date>

## Canvas (short)
| Context | Purpose | Class. | Decides on its own | Receives | Sends | Own words |
|---|---|---|---|---|---|---|

## Scenarios (short)
| # | Case | Contexts | Crossings / round trips | Leads to |
|---|---|---|---|---|

| Likely change | Contexts touched |
|---|---|

## What holds
- <part of the cut> — <which test it passed>

## Findings
### F1 <one-line claim>
- Severity: blocking | should fix | note
- Lens: language | ownership | dependencies | scenario | pattern
- Evidence: <element or file>: "<quote>"
- Why it matters: <consequence, concretely>
- Alternatives: <option> — costs <…>; <option> — costs <…>
- Level and next step: business (question) | architecture (proposed decision) | review

## Questions for the people who decide
1. <question that only the business or the maintainer can answer>
```

Severity: **blocking** means later work would build on it and become
wrong (a cycle, a rule with two owners). **Should fix** means the cut
holds but a word or mapping is misleading. **Note** means a risk without
a concrete failure yet.

## Not in scope (yet)

- Sharpening the language *inside* one context while modelling it, and
  the cut of aggregates (invariants, consistency boundaries). Both belong
  to the detailed modelling. Add them here as a second mode then, not as
  guesswork now.
- Whether the example data in the flows is consistent: that is
  `check-flow-data`. If a data error is also evidence for a finding about
  the cut (a rule that the data breaks at a boundary), mention it in that
  finding and point to `check-flow-data` for the rest.
- A missing strategic classification (core, supporting, generic) is an
  empty field in the canvas and, at most, a note.
