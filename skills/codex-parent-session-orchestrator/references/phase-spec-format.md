# Parent-Session Run Kit Format

The parent-session workflow uses a manual Markdown run kit instead of an external scaffolding script.

Recommended directory:

```text
parent-session-runs/<run-id>/
|-- task-brief.md
|-- active-context.md
|-- phase-checklist.md
|-- session-summary.md
`-- phases/
    |-- scan.md
    |-- plan.md
    |-- implement.md
    |-- verify.md
    `-- review.md
```

Add `fix.md`, `re-verify.md`, and `re-review.md` only when a material issue opens a bounded repair loop.

## `task-brief.md`

Capture:

- request summary
- constraints
- acceptance criteria
- known files or touch points
- risks worth tracking from the start

## `active-context.md`

Capture:

- current repository facts that matter right now
- open assumptions
- current target files
- current validation plan

Treat this as a bootstrap snapshot, not as the authoritative long-lived checkpoint once work begins.

## `phase-checklist.md`

Capture:

- ordered phases
- completion marks
- pause points
- any explicit return edges already known

This is the ordered phase index, not the durable current-state ledger.

## `session-summary.md`

Capture:

- accepted facts
- current phase
- approved fix scope, if any
- last completed validation or review result
- next action
- unresolved risks

This is the durable current-state authority.

## `phases/*.md`

Each phase file should contain:

- phase name
- role
- mode
- goal
- files to read first
- writable scope
- outputs to update on disk
- success criteria
- validation checks when needed
- stop condition
- selected imported skill paths when the phase needs vendored execution discipline

## Templates

Use the files under `assets/run-templates/` when you want a deterministic starting point without any helper script.
