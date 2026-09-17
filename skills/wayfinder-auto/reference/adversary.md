# The adversary

With the human gone, the only thing standing between a map and a model agreeing with itself is an
independent review. It has to be **structurally** blind — not blind by instruction. An agent that
hands a reviewer its own draft and its own reasoning has built a mirror.

## Assemble the workspace

The reviewer gets a directory you built for it, never the repo. A detached sparse worktree gives it
real source to cite while leaving out everything else — untracked files (`.env` and friends) are
absent by construction, and only the paths you name are checked out:

```bash
WS="$(mktemp -d)/review"
git worktree add --no-checkout --detach "$WS" HEAD
git -C "$WS" sparse-checkout set --no-cone <paths the question touches>
git -C "$WS" checkout
```

Then **verify** before you invoke anything — assert that the effort directory, any database,
any env or credential file, and any customer data are absent from `$WS`. If a forbidden path is
present, fix the path list; do not proceed and hope.

The current ticket's draft is uncommitted, so `HEAD` does not contain it. Already-resolved tickets
may be visible; that is fine and useful — decided is not the same as under review.

Remove the worktree when the review is done: `git worktree remove --force "$WS"`.

## Two phases, at most two calls

**Phase 1 — without the draft.** Give it the question, the constraints, the destination, and the
evidence paths. Ask what the real alternatives are, which requirements are missing, and what
evidence would falsify the premise. A reviewer shown the answer first will grade that answer
instead of the question.

**Phase 2 — with the draft.** Reveal the draft and ask where it is wrong, not where it could be
polished. Require a file and line, or a command and its output, behind every claim.

Review the destination itself once per map, and any research finding a decision rests on.

## Invocation

```bash
codex exec --sandbox read-only -C "$WS" --skip-git-repo-check \
  -c model_reasoning_effort="high" -o "$OUT" "$(cat "$PROMPT")" < /dev/null
```

- `-c model_reasoning_effort="high"` — the default is `none`, which produces a shallow review.
- `< /dev/null` — without it, an inherited open stdin leaves it waiting on input collection.
- `$OUT` and `$PROMPT` live in the same per-run `mktemp -d`; never a fixed path, or two efforts
  collide on the same ticket number.

Preflight: the binary exists and is authenticated. Postflight: the process exited zero within a
bound you set, and `$OUT` is fresh and non-empty. Any of those failing means the ticket is
**parked**, with the draft attached — never resolved on an unreviewed draft, never resolved on the
assumption that silence was agreement.

No `codex` on the machine is not fatal: run the same two phases as a subagent on a different model
and label the review as weaker in the answer. What is fatal is skipping it.

## Reconcile

One line per finding: **accept** (rewrite the decision — accepting a caveat is not accepting a
finding), **partial**, or **reject**, each with its reason. Accepting every finding is a legitimate
outcome; so is rejecting every finding. Do not manufacture disagreement to look rigorous.

After the second call, a material objection still unanswered by evidence means the question is not
ready: park it `needs-human` with both positions and the evidence that would settle it. Split a
ticket only when you can name the independently answerable subquestions — splitting to reset the
review budget is how a map grows without deciding anything.
