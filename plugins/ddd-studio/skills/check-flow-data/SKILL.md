---
name: check-flow-data
description: Check that the example data in a DDD Studio flow tells one consistent story - identities, times, quantities, invariants, policy conditions and read-model rows across all slices. Use when a flow has example data, before slices are marked as test scenarios, or when asked to "check the flow", "check the example data", "do the examples make sense" or "review the scenarios". Reports contradictions with evidence; it never changes the model.
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
- **Report, do not repair.** Change nothing. For each finding propose the
  smallest correction (which value to change, or that the slice should be
  a rejection instead) and say if more than one correction is possible;
  the modeller decides which value is wrong.
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
  glossary defines a word the data uses. Commands and events only when
  they have constraints; actors and queries without parameters not at
  all. Fewer calls keep the check fast enough to run live,
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
4. **Classify** each finding: **contradiction** (two values cannot both be
   true), **rule broken** (data breaks an invariant, constraint, policy
   condition or projection rule the model states), **suspicious** (likely
   wrong, but the model does not say so), **gap** (the story implies a
   step or value that no slice shows).
5. **Report** (format below) in the conversation. On the user's go, add
   each finding as a note under its slice in Studio (`create_note`,
   canvas `{ view: "flow", flowId, sliceId }`, targets: the member nodes),
   so the modeller sees it where the data is.

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
