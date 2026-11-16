# Gates

A gate is a sign-off by a named human at the end of a phase. It is mandatory, and the
next phase does not start without it. A checkpoint is a smaller stop inside a phase for
a decision or an RFI; the agent asks and continues with unrelated tasks while it waits.

## Requesting a gate (agent)

1. Run every command in the AGENTS.md Commands table, including the scaffold check.
   Save raw output under `artifacts/phase-N/` and add one line per check to the
   STATUS.md Evidence table.
2. For each exit criterion of the phase in `docs/acceptance.json`, run its `verify`
   step. Set `passes` to true and fill `evidence` only for those you saw succeed.
   Verify end to end, the way a user would, not only with unit tests.
3. Run the fresh-context review in `review.md`. Fix findings that affect correctness or
   the stated requirements, then re-run the affected checks.
4. Make sure `CHANGELOG.md` `[Unreleased]` lists the phase's user-visible changes.
5. Work out the proposed version from `versioning.md`.
6. Write the gate request block below into STATUS.md and set `Gate: GN REQUESTED`.
7. Commit locally: `chore(gate): request G<N>`.
8. End your turn with: "Phase N complete. Gate GN awaiting approval from <names>.
   Needed from you: <list>."

If an exit criterion cannot pass, say so in the request. Do not request a gate by
omitting it.

### Gate request block

```
## Gate request

Gate: G<N>, phase <N> (<name>)
Approvers: <names from docs/PLAN.md>
Exit criteria: <ids> all passing | failing: <ids and why>
Review: <one line: findings fixed, findings left and why>
Proposed version: <X.Y.Z or none> (<major|minor|patch>: <reason from commits>)
Look at: <files, test output paths, screenshots>
Open decisions or RFIs the next phase depends on: <ids or none>
```

## Approving or rejecting (human)

Add one line to the Ledger section of `GATES.md`:

```
G2: APPROVED by Ada Example, 2026-01-15. Version: 0.3.0. Notes: ship it
G2: REJECTED by Ada Example, 2026-01-15. Notes: retry logic untested
```

Use `Version: none` to close a gate without a release. After a rejection the agent
addresses the notes and requests the gate again.

## Closing a gate (agent, after approval)

Start by reading `GATES.md`. If there is no APPROVED line for the gate, stop.

1. If the approved version is not `none`: set `VERSION`, set every version mirror for
   the profile, turn `[Unreleased]` in `CHANGELOG.md` into `[X.Y.Z] - YYYY-MM-DD` with a
   fresh empty `[Unreleased]` above it, and update the README version badge. Use the
   version the human approved, even if it differs from the one proposed.
2. Update the README per `readme.md`: features, quick start, roadmap status, phase
   badge.
3. Compact per `context.md`: archive the phase, summarize its PLAN.md section, move
   retired decisions and RFIs.
4. Rewrite `STATUS.md` for phase N+1: new phase number and name, next tasks, empty
   evidence table, `Gate: G<N+1> not requested`.
5. Run `python3 scripts/check_scaffold.py`. It must pass.
6. If a version was released, give the human the tag commands to run once the push
   succeeds:

   ```
   git tag -a vX.Y.Z -m "vX.Y.Z"
   git push origin vX.Y.Z
   ```

7. Do not commit the changes yourself. As the last action of your turn, invoke the
   `yeet` skill with the summary `chore(release): vX.Y.Z (close G<N>)`, or
   `chore(gate): close G<N>` when there is no release. It asks the human to confirm,
   then commits these changes and pushes the whole phase.

Invoke `yeet` last because it may switch the session to a smaller model for the rest of
the turn. If the skill is unavailable or the harness blocks it, ask the human to run
`/yeet`; do not reproduce its push with other commands.

The next phase starts in a new session.

## What CI enforces

`scripts/check_scaffold.py` fails when the phase in STATUS.md is ahead of the approved
gates, when an approved gate has failing criteria, when `VERSION` differs from the last
approved version, or when the ledger is malformed. It cannot tell who wrote a line. To
enforce that only approvers change `GATES.md`, protect the default branch and list the
file in `.github/CODEOWNERS`.
