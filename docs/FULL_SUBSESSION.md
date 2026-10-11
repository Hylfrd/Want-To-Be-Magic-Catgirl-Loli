# Full subsession coordination protocol

Apply only when the user explicitly enables this mode, subject to the root `AGENTS.md` rules for scope, verification, AI records, and Git. `FULL_SUBSESSION.zh-CN.md` is the Chinese review translation.


### Activation and roles

- Activate only on an explicit `full subsession` or `full session` request for one root task. Policy discussion/editing does not activate it. Only Main owns the user's root `/goal`.
- Create delegate chats in the existing project directory using the local environment, never worktree/branch setup. Activation requests separate sidebar-visible delegate chats and coordination messages within tool permissions. No built-in/hidden subagents. This overrides Markdown delegation defaults, not higher-priority/tool rules; report unavailable tools without substituting topology.
- Main/Submains use `gpt-6.1-sol` / `xhigh`; implementation, integration, and review Subsessions use `gpt-6-luna` / `max`. Preserve Main's identity/settings. Assume the configured 500k windows; do not fill them needlessly.
- Main plans medium tasks, quotas, dependencies, and root acceptance; it manages Submains. Submains plan small tasks and manage their Subsessions. Coordinators normally do not implement product code.
- After two failed repair rounds for one issue, Submain reassesses and may make a bounded repair; record the exception. Main may perform deterministic Git integration; delegate code/conflict repairs. All commits go directly to existing `main`; only Main coordinates serialized Git writes and remote pushes. No independent child pushes or new branches/tags/issues/PRs or workaround worktrees.

### Local state and context recovery

- Before dispatch, persist in ignored `runtime/full-session/<goal-id>/`. No database, daemon, orchestration framework, CI gate, or telemetry system for this mode.
- Use `goal.md` for the original goal, exclusions, acceptance, mode, identities/state path; `plan.md` for medium tasks/dependencies; `overview.md` for concise status; `medium/<id>.md` for small-task plans; `results/<id>.md` for handoffs; `manual-tests.md` when needed.
- Main owns root state, quotas, and registry; Submains own medium plans; workers own results/logs. One writer per file. Supply one explicit `state_root` in the existing checkout to every chat. Recover missing state from the parent, never invent it.
- At startup, every new turn, after interruption/compaction, and before dispatch/integration, read relevant state: role, task/parent/root IDs, mode, scope, quota, dependencies, next action. Never rely on compressed history alone.
- Repeat sidebar-only, role/model, scope limits, no child pushes, and cap 12 in starting prompts/recovery handoffs. Main reads its overview/relevant entries; other roles read their tasks/contracts. Avoid whole-plan, repository-map, history, and transcript reloads.

### Planning and concurrency

- Define all medium scopes/acceptance before implementation dispatch; define small tasks before dispatch. Each task records ID/requirement, outcome, owner, edit scope, inputs/contracts, dependencies, acceptance, rough duration, status, results/commits, blockers.
- Aim for medium tasks of about an hour and small tasks within one or two context windows. Estimates are approximate; split at delivery boundaries, not by exhausting context or abandoning work at a timer.
- Use a dependency graph; promptly dispatch ready independent work and prioritize tasks unlocking others. Do not invent work to fill slots or race dependent edits. Freeze goal/acceptance; record justified splits, reordering, or corrections within scope. Optional improvements need user authorization.
- Main controls reservations: `1 + Submains + Subsessions <= 12`, at most 4 Submains and 9 workers. Four Submains leave 7 worker slots; two leave 9. Assign quotas; Submains cannot exceed/transfer them independently.
- Reserve before launch, counting pending/starting, running, integration, and review sessions. Release only after confirmed completion/shutdown and saved handoff; unconfirmed stalled chats retain slots. Reconcile actual chats/file ownership/main commits before dispatch/retry. One implementation owner per task; requests, archive state, and local status files do not prove launch/shutdown/completion.

### Delivery and recording

- Workers use the existing `main` checkout with disjoint file ownership; serialize same-file edits. Submain requests a Git write window from Main and grants it to one worker for staging/committing. Stage only owned paths; unrelated staged work blocks the commit and must not be unstaged or included. No branches, tags, issues, PRs, worktrees, or detached commits for isolation; implementation and AI-record checkpoints are committed directly to `main`.
- Save detailed outputs as files/commits; pass original references downstream. Workers report to Submain. Submain contacts Main for medium delivery, cross-task dependencies, quota changes, unresolved blockers, or user decisions; keep repair exchanges, code, and full tool output out of Main.
- Medium handoffs contain ID/status, delivered requirements, base/final main commits, exact task-owned commit lists, evidence paths, remaining checks/blockers, and needed action. Load details only for a necessary decision.
- Commits are submitted, not accepted. Review the agreed contract at the exact commit and verify combined medium behavior. Recheck affected portions after changes; interleaved main commits from other tasks are not this task's commits. If relevant files change during checks, wait for their writers and identify the verified version rather than create a test branch. Reviewers separate blockers/suggestions, obey root `AGENTS.md` Section 0, and stop when acceptance is met.
- Main checks every original requirement against evidence, delegating final integration/acceptance through Submains. Accepted checkmarks cite evidence; executor claims alone do not suffice.
- Log all Submains/workers and delegated recovery replacements under `ai-logs/subagent/` with immediate parent/root identities. Follow root `AGENTS.md` Sections 1-4 for original dialogue and separate commits. Export/deduplicate session sources without printing transcripts into Main; return paths, coverage, gaps. Runtime summaries never replace dialogue.

### Liveness monitoring, stalled-session recovery, and archiving

- Use existing coordinators for mutual monitoring, not an extra watchdog agent, daemon, or telemetry system. Main monitors Submains; each Submain monitors its workers. Main designates one existing Submain as the sole Main recovery monitor; other Submains report Main anomalies to it. Without an available Submain, automatic Main recovery is unavailable.
- At dispatch, record task-appropriate check intervals, inactivity timeouts, and expected long tool operations. Check app-native status, last events/output, actual tool/process activity, and file/Git progress. Prefer compact cursor-based waits/snapshots; do not flood Main with full histories or chatty heartbeat messages.
- After repeated lack of progress beyond the expected quiet interval, rule out normal lengthy reasoning/tools, waiting for user/approval, and explicit pauses; then send one brief authorized status probe. Confirm a stall only after continued nonresponse and absent execution progress. Silence or busy/idle labels alone do not prove failure; this protocol does not diagnose its cause.
- For a confirmed stall, stop waiting indefinitely or sending the same prompt in a loop. Preserve commits, uncommitted work, task state, and available original dialogue, marking interruption/gaps. Stop the old chat through supported tools and verify shutdown, then create a fresh chat with the same role/model and durable task to resume. Do not terminate a progressing long tool operation or run old and replacement executors concurrently on the same task.
- Only the designated recovery monitor coordinates Main replacement. After old Main is stopped and handoff saved, retain the logical Goal ID, original root conversation ID, objective, acceptance, and child relationships; update the active Main ID and replacement record. New Main resumes the same objective: restart is not Goal completion or a second parallel root task. Report inability to restore native Goal state reliably.
- If tools cannot stop/restart a chat or verify shutdown, retain its slot, isolate conflicting dispatch, and ask the user to perform the missing stop step. Archive is not shutdown; do not kill the entire app or unrelated processes as a substitute. Explicit user pause/stop/cancel forbids automatic restart.
- Promptly archive completed, obsolete, and replaced chats once they have no remaining work, decision, or recovery duty and their work/handoff/logs are saved. Keep chats needed for the next stage. Archive replaced chats after verified shutdown and recording; preserve needed deliverables and task files.

### Blockers and completion

- Preserve commits, unfinished changes, failed approaches, blockers, and next actions on stalled/repeated failures. Continue independent work and escalate. No unchanged polling, endless recreation, or silent omission of required tasks.
- Replace workers only after understanding remaining work, saving handoff, and confirming shutdown; archive is not cancellation. Preserve needed task files/changes. If tools cannot stop a chat, report it and retain its slot. New workers receive focused handoffs, not entire parent histories. Replace Main only for a confirmed stall under the recovery procedure above, not merely to avoid compaction.
- Pending device checks record build/commit, prerequisites, actions, expected results, missing evidence. Finish independent work; consolidate the manual list for Main.
- Complete the root Goal only with original acceptance evidenced, owned work/records committed, and no required task/decision left. Await manual results if required; an agreed testable-delivery Goal may finish with explicit pending device acceptance.
- Reconcile children and preserve evidence before archiving chats/retiring temporary state. Completion does not authorize maintenance, monitoring, or extra improvements.
