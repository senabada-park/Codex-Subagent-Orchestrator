# `/sub` Command Protocol

## Purpose

This protocol defines how the parent Codex instance should interpret requests that begin with `/sub`.

## Trigger Rule

When the user message begins with `/sub`, treat it as an explicit request to enter supervisor mode and delegate execution to one or more `codex exec` workers.

Interpret:

```text
/sub <request>
```

as:

- orchestration is required
- direct parent-only execution is not the preferred path
- the parent should form and supervise a worker team

## Parent Actions

When `/sub` is used, the parent should:

1. strip the `/sub` prefix
2. interpret the remaining text as the true task request
3. decide whether the task needs one worker or a team
4. choose worker roles autonomously
5. choose model, sandbox, and reasoning per worker
6. launch and supervise the workers
7. validate outputs before reporting back
8. preserve enough evidence for later review: manifest, prompt files, worker summaries, and a per-run archive with worker-specific folders
9. if the parent recovers from a wrong delivery path or wrong workspace root, run the reviewer or verifier again against the final successful artifact before accepting it
10. if a reviewer finds a material issue, launch a bounded fixer worker instead of patching the deliverable directly in the parent
11. when building a launcher spec for deliverable work, include `requested_deliverables`, `supervisor_only: true`, `require_final_read_only_review: true`, and `material_issue_strategy: "fixer_then_rereview"` so unsafe team shapes fail fast

## Team Sizing Rule

The team should be chosen autonomously.

Use:

- `1` worker when one bounded worker can finish the task cleanly
- `2` workers when one worker should implement and one should verify, or when two outputs are cleanly independent
- `3` workers when planning, implementation, and review should be separated
- `4+` workers only when the work is truly parallelizable and file-level merge risk remains manageable

Do not add workers just because `/sub` was used. Add workers only when team structure improves execution quality or throughput.

## Worker Contract Rule

Every worker should receive:

- the shared operating contract from workspace `AGENTS.md` when available
- a role-specific mission from the parent
- a bounded task definition
- explicit writable scope
- validation instructions
- a return contract

Reviewers and validators should default to `read-only` unless a narrower exception is explicitly justified.

## Efficiency Rule

`/sub` does not mean "use maximum reasoning everywhere."

The parent should adjust reasoning efficiently:

- `low` for routine execution workers
- `medium` for moderate ambiguity
- `high` only when the worker's decision burden is genuinely complex
- `xhigh` only for exceptional cases

The same rule applies to model choice. Use the cheapest model that still safely fits the worker's task.

Do not evaluate orchestration quality by absolute token totals alone.

Prefer structure-first efficiency signals:

- keep parent intervention small
- avoid unnecessary full reruns
- prefer reviewer -> fixer -> reviewer loops over rerunning the whole team
- keep worker count proportional to requested deliverables
- use token totals only as a secondary comparison signal

If the parent intentionally chooses a model, it should pass `-m` explicitly so the run is reproducible.

## Parallel Rule

Run workers in parallel only when:

- they are independent
- they do not compete for the same writable scope
- the parent can merge results deterministically

When the launcher is used, prefer same-stage parallel workers plus a later-stage reviewer or validator instead of putting writers and reviewers in the same stage.

If those conditions are not met, use staged execution instead.

## Reporting Rule

The parent remains the final reporting authority.

Workers produce bounded outputs. The parent integrates, validates, and reports the final result.

When the launcher path fails, preserve the failed spec and fallback reason, then pivot cleanly to direct `codex exec` rather than silently switching behavior.

If the parent pivots to direct `codex exec`, it should preserve the intended worker settings explicitly:

- pass `-m` when the model choice matters
- pass `-c 'model_reasoning_effort="low"'` for routine workers unless the task risk justifies more
- keep `-o` so the final worker message is preserved on disk
