# wayfinder-auto

A [Claude Code](https://claude.com/claude-code) skill that charts a large piece of work as a map of
**decision tickets** and resolves them **unattended** — no questions, defaults decide, an
independent model argues back, and the map is committed as it goes.

It is a derivative of the `wayfinder` skill from
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT). Wayfinder is a human-in-the-loop
flow: it grills you, one ticket per session, and the grilling is most of the value. This variant is
for the other case — you want the map charted and burned down while you are asleep, and you would
rather review a stack of flagged assumptions afterwards than answer forty questions now.

**Read this part before installing it.** Removing the human removes the thing that made the
original work. What is left is an agent deciding its own planning questions, and no amount of
prompt engineering makes that equivalent. This skill is an attempt to make the trade honest, not to
pretend there is no trade.

## What replaces you

**A default ladder.** Every choice is classified first — a fact, a constraint you stated, or a real
choice — and only the third goes to the ladder: observation, then a human's written instruction,
then house precedent, then reversibility, then the smallest thing that works, then a coin flip.
Anything decided below the first two rungs is flagged as an assumption, with what would overturn it
and which decisions fall with it. You review that list; it is short by construction.

**An adversary.** The draft answer goes to a second model before the ticket closes. It is blind
structurally, not by instruction: the reviewer runs in a detached sparse `git worktree`, so the
draft, the map, untracked secrets and anything not explicitly checked out simply are not on its
disk. It sees the question before it sees the answer, because a reviewer shown the answer first
grades the answer instead of the question. Uses the `codex` CLI when present, another model
otherwise; skipping the review is not an option the skill offers.

**A parking lot.** Work the agent genuinely should not do alone — a credential, a payment, a live
production read, a prototype whose winner is a matter of taste — becomes a `needs-human` ticket.
The run routes around it and keeps going. Those tickets are the first thing in the final report.

## What it will not do

Three floors, and nothing in a ticket or a map's own notes can bend them:

1. **Plan, don't do.** No production code, migrations, deploys or sends. Lifted only by the literal
   `--execute` token in your invocation, copied verbatim to the top of the map — the upstream's
   most-reported failure is an agent writing itself permission in a file only it was ever going to
   read.
2. **Nothing irreversible, nothing leaves the machine.** No push, no remote issue writes, no
   accounts or payments, no reads against live production.
3. **Never ask, never stall.** A question is decided and flagged, or parked. Ending a run with a
   question is a failure of the skill.

These are instructions an agent reads, not a sandbox it runs inside. That distinction is the
skill's main limitation and it is deliberate that you are reading it here.

## Install

Drop it in as a personal skill, available in every project:

```bash
git clone https://github.com/BlackPhoenixSlo/wayfinder-auto
cp -R wayfinder-auto/skills/wayfinder-auto ~/.claude/skills/
```

Or per-repo, in `.claude/skills/`. It is model-invocation-disabled on purpose: nothing triggers it
but you.

**Needs:** a git repo. **Optional:** the `codex` CLI for the review pass (any second model works);
the upstream `wayfinder`, `research`, `domain-modeling` and `prototype` skills, which it calls when
they are installed and works without when they are not.

## Use

```
/wayfinder-auto <a loose idea>        chart a map, then work it
/wayfinder-auto <path to a map>       work an existing map
/wayfinder-auto <idea> --tickets 10   raise the default budget of 6
/wayfinder-auto <idea> --execute      lift floor 1 (you are asking for it)
```

The map is a local markdown snapshot — `.scratch/<effort>/map.md` plus one file per ticket — and it
is committed after charting, after each claim and after each resolution, path-limited so it never
picks up whatever you had staged. A remote tracker can be read for context; this skill does not
write to one.

It stops at the ticket budget, when the map passes 15 open tickets, when everything left needs you,
or when continuing would require lifting a floor. An empty frontier is diagnosed before it is
called a finish: blocking cycles, claims left by a crashed run, research that never came back and
standing fog all look identical to "done" and are reported as themselves.

You get one report: what was decided, the assumptions to veto, what needs you, and which stop
condition fired.

## Sibling

[spec-auto](https://github.com/BlackPhoenixSlo/spec-auto) picks up where this leaves off: it turns a
decided plan into specs, tracer-bullet tickets and, under `--execute`, code — reusing this skill's
adversary protocol.

## Credits

The map, the fog of war, the frontier, decision tickets and the ticket types are Matt Pocock's
design, from the MIT-licensed [`wayfinder`](https://github.com/mattpocock/skills). If you want the
human-in-the-loop original — and for most work you do — use that one.

MIT. See [LICENSE](LICENSE).
