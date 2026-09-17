# Artifacts

The map is a **local** snapshot, versioned in the repo. A remote tracker (GitHub issues, Linear,
Jira) may be read for context; auto mode does not create or update remote issues — floor 2 — so
record source links in the snapshot instead. If the repo documents its own tracker conventions,
follow those for layout and naming.

## Layout

Default, and what the upstream local-markdown convention specifies:

```
.scratch/<effort>/map.md
.scratch/<effort>/issues/NN-<slug>.md
```

**map.md**: `## Destination`, `## Notes`, `## Decisions so far`, `## Not yet specified` (the fog),
`## Out of scope`, `## Needs the human`, `## Assumptions to veto`. When the invocation carried
`--execute`, its literal text goes at the top of the map, quoted, with the date.

**Ticket**: the question in the body, plus header lines `Type:`, `Status:`, `Blocked by:`, and an
`## Answer` on resolution. Refer to tickets by title everywhere a human will read them; a wall of
numbers is illegible.

## Reading a map you did not write

Real maps drift from the spec. Normalise operational state, preserve the original text, and never
resolve an ambiguity in your own favour. Forms found in the wild, all of which must parse:

| Written as | Means |
| --- | --- |
| `Blocked by: 01` / `Blocked by: [20](20-slug.md)` | one blocker, bare or linked |
| `Blocked by: 02 (resolved), 06 (resolved)` | annotations are commentary — verify each blocker's own file |
| `Blocked by:` followed by prose | a stage gate; treat as blocked and flag it as unparsed |
| `Status: open` / `claimed` / `resolved` | the canonical set |
| `Status: **RESOLVED** 2026-07-31` | resolved |
| `Status: **CLOSED** 2026-08-01 — resolved, see Answer` | resolved |
| `Status: open — downgraded` | open, with a qualifier worth reading before claiming |
| `Type: domain-modeling`, `Type: grilling + prototype` | valid; hybrids run both ways |

Anything you cannot classify confidently is **parked**, not skipped and not assumed resolved. A
type you do not recognise is treated as `grilling` and flagged.

## Claims

`Status: claimed <run-id> <ISO-8601 UTC>`, written and committed before any work, so a second run
skips the ticket. A claim carrying a different run id older than a couple of hours is from a run
that died: reclaim it, and say so in the answer. Without that rule a crashed run poisons a ticket
permanently, because `claimed` is invisible to the frontier query forever.

## Commits

```bash
git commit -m "<prefix>(<effort>): <ticket title>" -- <effort dir>
```

- **Path-limited, always.** `--` scopes the commit to those paths and ignores the rest of the
  index. `git add <dir> && git commit -m …` commits whatever the user had already staged, which on
  a working machine is rarely nothing.
- Match the repo's own subject prefix — `git log --format=%s -- <effort dir>` tells you what the
  effort directory already uses.
- Commit after charting, after each claim, and after each resolution. A run that dies then leaves a
  readable history rather than a dirty tree.
- **Never switch branches in the user's working tree.** Research and prototype branches get their
  own `git worktree`; the tree you were invoked in stays as you found it.
- **Never push**, never open a PR, never touch a remote.

## An empty frontier is a diagnosis, not a finish

Before reporting a cleared map, check each of these, and report whichever fired by name:

- a blocking **cycle** — A blocks B, B blocks A, neither takeable;
- **claims from a dead run**, which look identical to work in progress;
- **research dispatched and never returned**;
- fog still standing in **Not yet specified**;
- destination criteria with no ticket covering them.
