---
name: check-flow-data
description: Check that the example data in a DDD Studio flow tells one consistent story - identities, times, quantities, invariants, policy conditions and read-model rows across all slices. Use when a flow has example data, before slices are marked as test scenarios, or when asked to "check the flow", "check the example data", "do the examples make sense" or "review the scenarios". Reports contradictions with evidence as notes under the slices concerned, with corrections as votable proposals; it never changes the model.
---

# Check the example data of a flow

`validate_model` checks that each example fits its member's schema: the
fields exist and have the right type. It does not check whether the
examples, read together, can all be true. A flow where the train number
changes halfway, where work ends before it began, or where a view still
shows a damage report as open after it was closed, validates clean. A
reader of the flow trusts those numbers, and a slice marked as a scenario
turns them into a test. This skill reads the flow as a story and checks
that the story holds.

The checks are in `checks.md` in this folder. Read it first.

## Rules

- **Evidence or silence.** Every finding names the slice, the member and
  the field, and quotes the values that contradict each other. A finding
  without two quoted values (or a value and the rule it breaks) is not
  reported.
- **The model is the measure.** Judge example data against what the
  model states: invariants, command constraints, policy constraints,
  projection rules, the ubiquitous language of the contexts, slice and
  flow descriptions. Do not invent rules from domain knowledge. Where
  your domain knowledge suggests a problem the model does not state,
  report it as **suspicious** or as a question, not as a rule broken.
- **One wrong value, one finding.** If one value breaks several checks
  (a time that is out of order and also breaks an invariant), report it
  once and list every check it breaks.
- **Report, do not repair.** Change no data. For each finding propose the
  smallest correction (which value to change, or that the slice should be
  a rejection instead) and say if more than one correction is possible;
  the modeller decides which value is wrong. The findings go under their
  slices as notes, the corrections as proposals (see "Publishing to DDD
  Studio").
- **Say what holds.** List the checks that passed, so that a clean report
  means something.

## Inputs

From DDD Studio (MCP tools `mcp__ddd-studio__*`):

- the workspace and the flow: if the user did not name them,
  `list_workspaces` / `list_flows` and ask; if the project's `CLAUDE.md`
  has a "DDD Studio" section, use the workspace it names,
- the names of contexts and nodes: `get_model_overview` (one call; the
  slices only carry ids),
- the flow's slices in order: `list_slices` with the flow id (name, type,
  order, members, description, exampleData, and, where set, trigger,
  consulted occurrences, rejection, answer). When a slice has a policy,
  event handler or process manager as a member but no trigger field, its
  trigger is the nearest event to its left that the member handles,
- the member nodes that carry rules: `get_node` on the aggregates
  (`identifiedBy`, invariants), the policies (`handles`, constraints) and
  the read models (projection rules), and on the bounded contexts whose
  glossary defines a word the data uses. Load a command only when its
  data looks wrong and you need to know whether a constraint allows it,
  and a query only when the slice description or an answer suggests a
  filter. Skip actors. Fewer calls keep the check fast enough to run
  live,
- the schema report: `validate_model` (run it first; schema warnings are
  reported separately and not repeated as findings),
- optionally the GWT features of the units in the flow: `list_features`
  (their given/when/then data can be checked the same way; an empty list
  is normal when no slice is marked as a scenario).

## Steps

1. **Lay out the story.** Write one row per slice in order: slice name,
   type, context, the members, and the example values that matter (ids,
   times, amounts, states). Note the slice description: it is the
   modeller's claim about what happens.
2. **Find the identities.** From `identifiedBy` and from fields that
   repeat across slices (an id, a number, a key), list the entities the
   story follows and the values each one has in each slice.
3. **Run the checks** in `checks.md`, in order: identity, command and
   event, time, invariants and constraints, triggers and policies, read
   models, quantities, references, description against data.
4. **Classify** each finding and list the checks it breaks (by their
   numbers in `checks.md`): **contradiction** (two values cannot both be
   true), **rule broken** (data breaks an invariant, constraint, policy
   condition or projection rule the model states), **suspicious** (likely
   wrong, but the model does not say so), **gap** (the story implies a
   step or value that no slice shows).
5. **Report** (format below) in the conversation, in the language of the
   user's request; the headings may be translated. Name slices by their
   name, not by a number.
6. **Publish** the findings as notes under their slices (next section),
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
  `Agent · check-flow-data · <kind>` (`<kind>` as listed under "What"). People see at once that an agent
  wrote it, and `clear-agent-notes` finds it again. Write everything after
  the marker in the language of the model.
- **Shape of a note.** After the marker: the claim in one sentence; then
  the evidence, quoted; then in one sentence why it matters. Keep it under
  about 600 characters: notes are read on a canvas, not in a report.
- **Few notes.** At most 8 notes per run. What does not make the cut stays
  in the report in the conversation.
- **Alternatives are proposals.** Put each alternative under its note with
  `add_comment` and `proposal: true`: what would change and what it costs,
  in two or three sentences. The people vote on proposals and decide.
- **No duplicates.** Before writing, `list_notes` on the same canvas. If an
  open note with this skill's marker already makes the same claim, do not
  write it again; say so in the report. If many notes of an earlier run are
  still open, ask the user whether to clear them first with
  `clear-agent-notes`.
- **Where.** Under the slice where the wrong value is: `create_note` with canvas `{ view: "flow", flowId, sliceId }` and targets of kind `node` (the members that carry the value).
- **What.** One note per finding (kind `Widerspruch`/`contradiction`, `Regel verletzt`/`rule broken`, `verdächtig`/`suspicious` or `Lücke`/`gap`), contradictions and broken rules first, at most eight. When the two values of a contradiction sit in different slices, put the note under the slice whose rule fails or whose value is most likely wrong, and name the other slice in the note. Under each, the smallest correction as a proposal; when two corrections are possible (which of two values is wrong), one proposal each.
- **Say what you published.** End the conversation report with the number
  of notes and proposals written, and where.

## Report format

```markdown
# Flow data check: <flow name>

Input: workspace <name>, flow <name>, <n> slices, read <date>
validate_model: <clean | n schema warnings, listed below>

## The story as the data tells it
| # | Slice | Context | Key values |
|---|---|---|---|

## Findings
### D1 <one-line claim>
- Kind: contradiction | rule broken | suspicious | gap
- Checks: <numbers from checks.md>
- Where: slice "<name>" (#n), member <node>, field <field>
- Evidence: "<value>" here, but "<value>" in slice "<name>" (#m) | breaks <invariant/constraint/rule>: "<quote>"
- Correction: <smallest change>; or <alternative>

## What holds
- <check> — <what was compared and found consistent>

## Questions for the modeller
1. <where the data is plausible but the model does not say what is meant>
```

## Not in scope

- Whether the cut into contexts is right: `challenge-context-cut`.
- Whether the payload fits its schema: `validate_model` does that.
