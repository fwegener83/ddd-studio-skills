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

All three share four rules: **propose, do not decide**; **evidence or
silence** (every finding quotes what it rests on); **say what holds**; and
**change nothing** in the model unless the user says so (and then only notes
or a new board, never the user's own material).

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
`/ddd-studio:challenge-context-cut` and `/ddd-studio:check-flow-data`, and
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

## Example

`examples/ice-werk/` describes a demo workspace: a fictitious overnight
shift in a high-speed train depot (German), with a Big Picture board, a
context map with three contexts and one flow. It is built to show what the
skills find.

## Origin

`challenge-context-cut` comes from the
[library-example](https://github.com/fwegener83/library-example) tutorial,
where it was written after use and tested by fresh agents; here it is
detached from that project's files.

## License

[MIT](LICENSE)
