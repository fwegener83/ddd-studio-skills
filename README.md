# DDD Studio skills

Agent skills for [Claude Code](https://code.claude.com) that work on a model
in [DDD Studio](https://ddd-studio.dev) through its MCP server. They do the
legwork around the decisions that are hard to undo, and leave the decisions
to people.

| Skill | What it does | When |
|---|---|---|
| `propose-context-cut` | Reads a Big Picture Event Storming board and proposes two or three alternative cuts into bounded contexts, each boundary backed by heuristics and the cards they rest on. | A Big Picture exists, the cut does not. |
| `challenge-context-cut` | Challenges an existing cut and context map: language, ownership of rules, dependency cycles, scenarios, mapping patterns. Findings with evidence and alternatives. | Before anything is built on the cut, and whenever it changes. |
| `check-flow-data` | Reads a flow as a story and checks that its example data can all be true together: identities, times, quantities, invariants, policy conditions, read-model rows. | Before slices become test scenarios, or whenever a flow's examples change. |
| `clear-agent-notes` | Removes the notes the other three published, and only those. | Before a demo or a fresh run. |

The first three publish their results in DDD Studio as **notes** where the
model is (on the board, on the context map, under the slice of a flow),
each starting with `Agent · <skill> · <kind>`. Alternatives go under the
note as **proposals** that people vote on with 👍/👎. The agent never
votes and never resolves a note: people decide. Ask for "report only" to
get the findings in the conversation without writing anything.

All of them share four rules: **propose, do not decide**; **evidence or
silence** (every finding quotes what it rests on); **say what holds**; and
**change nothing** in the model: notes and comments only.

## Install

The repository is a Claude Code plugin marketplace with one plugin,
`ddd-studio`.

For yourself:

```bash
claude plugin marketplace add fwegener83/ddd-studio-skills
claude plugin install ddd-studio@ddd-studio-skills
```

For everyone who works on a project, add this to the project's
`.claude/settings.json`; Claude Code offers the plugin when someone opens
the project and trusts the folder:

```json
{
  "extraKnownMarketplaces": {
    "ddd-studio-skills": {
      "source": { "source": "github", "repo": "fwegener83/ddd-studio-skills" }
    }
  },
  "enabledPlugins": {
    "ddd-studio@ddd-studio-skills": true
  }
}
```

The skills are then available as `/ddd-studio:propose-context-cut`,
`/ddd-studio:challenge-context-cut`, `/ddd-studio:check-flow-data` and
`/ddd-studio:clear-agent-notes`, and
Claude uses them on its own when a request matches.

The skills need the DDD Studio MCP server connected in the same session
(tools `mcp__ddd-studio__*`).

## Project conventions

The skills are generic: no project detail in their text. If a project wants
them to use a fixed workspace, read its written sources, or save reports in
a certain place, it says so in a "DDD Studio" section of its `CLAUDE.md`,
for example:

```markdown
## DDD Studio
- Workspace: "My Project", id `…`
- Sources: `requirements/brief.md`, decisions in `requirements/clarifications.md`
- Reports from challenge-context-cut go to `docs/reviews/<label>-context-cut.md`
```

## Examples

- `examples/demo/` describes three small demo workspaces, one per skill,
  with no notes in them, so that what the agent adds stands out.
- `examples/ice-werk/` describes a larger workspace with the same
  fictitious domain (an overnight shift in a train depot, German).

## Origin

`challenge-context-cut` comes from the
[library-example](https://github.com/fwegener83/library-example) tutorial,
where it was written after use and tested by fresh agents; here it is
detached from that project's files.

## License

[MIT](LICENSE)
