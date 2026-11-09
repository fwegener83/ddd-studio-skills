---
name: clear-agent-notes
description: Remove the notes that the ddd-studio skills (propose-context-cut, challenge-context-cut, check-flow-data) published in a DDD Studio workspace, to reset a demo or before a fresh run. Use when asked to "clear the agent notes", "reset the demo", "Agenten-Notizen löschen" or "aufräumen". Deletes only notes whose first line carries the agent marker; never touches notes written by people.
---

# Clear agent notes

The other skills of this plugin publish their findings as notes. A saved
version in DDD Studio does not include notes, so restoring a version does
not remove them. This skill does, and nothing else.

## Rules

- **Only agent notes.** Delete a note only when its body starts with
  `Agent · ` followed by the name of one of the skills of this plugin
  (`propose-context-cut`, `challenge-context-cut`, `check-flow-data`), and
  its author shows it was written via an agent (`via: "mcp"`). A note that
  fails either test stays, even if it looks like an agent's. `via: "mcp"`
  alone means little (any tool can write through MCP); the marker is the
  real filter.
- **Answers make it a conversation.** If people have commented on or voted
  in an agent note, it is no longer only the agent's. List those notes and
  ask before deleting them, unless the user said to reset everything.
- **Say first, then delete.** Before deleting, tell the user how many notes
  you will delete, per canvas, and wait for a yes. Deleting cannot be
  undone. If the user already said to go ahead, announce the counts and
  delete in the same turn.
- **Nothing else changes.** No cards, nodes, mappings, slices or versions.

## Steps

1. Find the workspace: use the id if given, otherwise `list_workspaces`
   and ask.
2. `list_notes` for the workspace (open and resolved). Keep the notes that
   pass both tests under "Only agent notes". Where `commentCount` or
   `affectsMeCount` is above zero, people have answered: read it with
   `get_note` (comments, votes on proposals).
3. Tell the user what will be deleted: the count per canvas (board, context
   map, flow), and the notes with comments or votes, if any.
4. Delete with `delete_note`, one by one.
5. `list_notes` again and report: deleted, kept (with the reason), and
   that the remaining notes are not from this plugin's skills.

If the user restricts it ("only the flow notes", "only check-flow-data"),
apply the restriction to the list in step 2.
