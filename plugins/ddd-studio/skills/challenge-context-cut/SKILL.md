---
name: challenge-context-cut
description: Challenge the cut into bounded contexts and the context map in DDD Studio before anything is built on it. Use at the end of strategic design, before work that builds on the cut, after a context is added, merged or split, after a mapping is turned, or when asked to "challenge", "stress-test" or "review the domain cut / context cut / context map". Produces findings with evidence and alternatives; it never changes the model.
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
- **Propose, do not decide.** Change nothing in Studio or in the repo
  except the report. Every serious finding comes with at least one
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
   questions in the conversation. Save the report as a file only where
   the project says (its `CLAUDE.md`) or the user asks; a common place is
   `docs/reviews/<label>-context-cut.md`. On the user's go, add each
   question as a note on the context map in Studio (`create_note`,
   canvas `context-map`, targets: the contexts or mappings concerned).

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
