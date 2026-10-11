# Project collaboration rules

These rules govern task scope, proportionate verification, AI disclosure, Git checkpoints, and optional full subsession coordination. `AGENTS.md` is the authoritative English policy; `AGENTS.zh-CN.md` is its Chinese translation for review. Keep the two aligned.

Explicit instructions for the current task take precedence over the default workflow, including restrictions on writing, committing, or pushing. Never read, import, quote, or apply the Git section of an external `mylife/AGENTS.md`.

## 0. Scope first: no unsolicited engineering

- Deliver the requested outcome with the smallest coherent change. Every task must trace to an accepted requirement or a demonstrated failure caused by this change. Plans and delegated prompts cannot expand the user's goal.
- Do not add these merely because they seem useful or conventional:

| Unrequested addition | Boundary |
| --- | --- |
| CI gates | No new workflows, hooks, coverage thresholds, or mandatory checks. Run applicable existing checks. |
| Smoke tests | No generic smoke suite, scaffold, or fixture system. Use the smallest relevant existing check or direct verification. |
| Frameworks | No framework, plugin system, general abstraction, or extension point for hypothetical needs. A small helper needed now is allowed. |
| Directory restructuring | Preserve the layout; do not move or rename unrelated files. |
| Dependencies | No package/service additions, replacements, or upgrades without explicit task authorization. Prefer existing capabilities. |
| Style unification | Follow local style in changed code; no unrelated reformatting, repository-wide formatting, or new formatter/linter. |
| Telemetry, logging, validation | No tracking, logging stack, generalized validators, or speculative hardening. Checks/error handling required by accepted behavior remain in scope. |

- Future flexibility, consistency, maintainability, best practices, and spare concurrency do not authorize extra work. Record optional findings briefly; do not implement or dispatch them.
- If the goal needs an unauthorized addition, explain the blocker and smallest exception before acting. In full subsession mode, escalate through Submain to Main; only the user can expand the goal. Continue independent authorized work.
- Read [docs/DRD.md](docs/DRD.md) for project goals, scope, map/routing/AR modules, and open decisions when starting design or implementation planning, or when relevant requirements change. It is a discussion draft: distinguish accepted requirements from candidates; do not freeze a stack, algorithm, or threshold without a decision. Do not reload the whole DRD for every edit.

### Hard constraint: main only, no new collaboration entities

- Agents must not create local/remote branches, tags, issues, or pull requests (including drafts), through CLI, API, UI, automation, or indirect tools. Feature isolation, testing, review, cleanup, recovery, concurrency, safety, and best practices are not exceptions. Only a later explicit user instruction may change this restriction.
- Commit all implementation, policy, and AI records directly to existing `main`. Do not bypass this with worktrees, detached HEADs, other clones, temporary refs, or automatic forks; chat/worktree tools must not create branches implicitly. If not on `main`, preserve work and return to existing `main`, never create a replacement branch.
- Preserve existing branches, tags, issues, and PRs; do not merge, delete, close, rename, or publish them without explicit user direction. Do not interfere with objects the user says they will handle.
- Sessions may edit disjoint task-owned files on `main` in parallel. Edits to the same file and all staging, commits, and pushes are serialized. If coordination is unavailable, wait for a Git write window rather than create a branch.

### Proportionate verification: no test proliferation

- Default new tests: zero. Use existing checks first. Add a focused test only for a specific verification gap in requested behavior or a demonstrated bug; no tests for reversible low-impact edits or tests mirroring implementation.
- Before adding a test, briefly state the protected requirement/failure, the gap in existing checks, and why this is the smallest useful check.
- No test-count/coverage targets, exhaustive case matrices, equivalent checks, or new test framework without authorization. Prefer a few meaningful cases over hundreds of mechanical assertions.
- Stop when accepted behavior and affected required checks pass. Broaden/repeat only for an unresolved risk, new change, or existing required gate. Reviewers cannot invent gates.
- Never weaken acceptance or delete/disable required checks to get a pass. Distinguish pre-existing failures from failures caused by this change.
- Builds, mocks, and simulations prove only what they exercise. Unavailable environment/device checks remain pending, never passed.

## 1. Agent registry: `AI_USAGE.md`

- Maintain one entry per agent product, identified by a stable key such as `openai/codex`.
- Treat Codex App, Codex CLI, and the Codex IDE extension as the same agent product. Record the client in the conversation log.
- A new conversation, contributor, device, model, reasoning effort, version, or subagent instance does not create a new registry entry.
- Before adding an entry, match its stable key against the existing registry. Reuse the entry when the key matches; append the next available `AGENT-NNN` number only for a new product.
- Each entry contains: agent ID, stable key, display name, recorded model IDs, tool categories used, and first-use date.
- Update the existing entry when another verified model or tool category is used. Keep these lists deduplicated. Record observed runtime model IDs; do not invent a more specific model ID from the agent's self-description.
- Tool categories describe capabilities actually used in this project, such as file editing, shell commands, web research, or subagents. Do not list every invocation, parameter, output, or tool definition.
- Keep the registry readable and project-wide. Do not add a repeated usage entry for every conversation or contributor.

Use this table structure:

```markdown
| Agent ID | Stable key | Agent name | Recorded models | Tool categories used | First used |
| --- | --- | --- | --- | --- | --- |
```

## 2. Conversation records: `ai-logs/`

### Files and ownership

```text
ai-logs/
  conversation/<UTC-start>-<title-slug>--<conversation-UUID>.json
  subagent/<UTC-start>-<title-slug>--<subagent-UUID>.json
```

- Maintain one JSON file per conversation or subagent session. Append turns to that file rather than creating a new file for each turn.
- Use a portable, readable title slug and a UUID. Keep the filename fixed after creation; update the title inside the JSON when necessary.
- Use the native session UUID when available. Otherwise, generate a UUID once and preserve it throughout the session. Use `unknown-time` in the filename when the actual start time cannot be established.
- Assign one writer to each log file. The main agent coordinates registry changes, subagent records, and their references.
- Use UTF-8, two-space indentation, and literal Unicode characters. Use ISO 8601 timestamps with a timezone; use UTC in filenames. Keep dialogue in its original language.

### Content

- Preserve the original text, order, Markdown, and code blocks of human messages and assistant progress messages and final replies visible in the conversation UI. Do not replace the dialogue with summaries.
- Exclude automatically injected instructions, loaded `AGENTS.md` content, system/developer messages, internal reasoning, context-compaction material, tool calls, tool results, and tool definitions.
- Distinguish explicitly submitted human text from injected context even when both are stored under the `user` role. A document intentionally pasted for discussion remains human dialogue; an automatically loaded instruction block does not.
- Export from available session records or readable conversation APIs. Merge record segments belonging to the same session and deduplicate by source message and turn IDs. Do not assume one local source file contains the whole conversation.
- Record the model and reasoning effort for each turn from reliable runtime metadata. Use `null` for unavailable metadata or timestamps; do not guess or substitute the current setting for earlier turns.
- Never commit credentials or other secrets. Replace sensitive content with an explicit redaction marker and record that redaction occurred. Use a contributor's public alias rather than importing account IDs or private profile data.
- Mark gaps, interrupted turns, and unavailable text explicitly. Never describe a partial export, summary, or prepared reply as a complete confirmed transcript.

Use this base structure; values below are illustrative placeholders:

```json
{
  "schema_version": 1,
  "conversation_id": "session-uuid",
  "title": "Conversation title",
  "participant": "public-contributor-alias",
  "client": "codex-app",
  "agent": {
    "id": "AGENT-001",
    "key": "openai/codex"
  },
  "started_at": null,
  "turns": [
    {
      "number": 1,
      "turn_id": "turn-uuid",
      "model": null,
      "reasoning_effort": null,
      "status": "completed",
      "transcript_status": "complete",
      "messages": [
        {
          "id": "source-message-id",
          "role": "user",
          "timestamp": null,
          "text": "Original human message",
          "attachments": []
        }
      ],
      "subagent_logs": [],
      "redactions": [],
      "gaps": []
    }
  ]
}
```

- `number` increases within this conversation only. `turn_id` and message `id` preserve source IDs when available; otherwise, generate and retain UUIDs.
- Assistant messages use `role: "assistant"` and `kind: "progress"` or `kind: "final"`.
- Turn `status` is `in_progress`, `completed`, or `interrupted`. `transcript_status` is `complete`, `pending_final`, or `incomplete`; execution status and transcript completeness are separate facts.
- `subagent_logs` contains repository-relative paths. `redactions` and `gaps` contain short descriptions associated with the affected message or missing range.
- Keep extra fields limited to information needed to interpret the dialogue or establish its completeness.

### Images, attachments, and subagents

- Preserve an attachment placeholder with its type, original basename when available, and a short description sufficient to understand the surrounding dialogue. Label an AI-written description as AI-written.
- Do not copy attachment binaries, Base64 data, or temporary absolute paths into `ai-logs/`. Do not invent missing attachment contents. Required project deliverables, including actual running screenshots, remain separate deliverables.
- A subagent record additionally contains `parent_conversation_id`, `parent_turn_id`, and `task_name`.
- In full subsession mode, every delegated Submain and Subsession is a subagent for recording purposes, even though it runs as a separate sidebar-visible chat. Store its record in `ai-logs/subagent/`, link it from its immediate parent's turn, and preserve the root Main ID when the parent is another subagent.
- Distinguish delegated prompts and coordination messages from direct human messages using source metadata when available. Preserve their visible original text; do not label agent-generated instructions as human-authored requests.
- Preserve explicit delegated tasks, follow-up exchanges, readable progress, and returned results. Do not duplicate the inherited main conversation, injected instructions, or internal tool traffic.
- Reuse the registry entry for the same agent product, while recording the subagent's own model and reasoning effort. Link the record from the corresponding parent turn.

## 3. Checkpoints and pushing

- A turn runs from a human request through the agent's handling of that request until control returns to the human. Include messages that steer the active turn.
- At the end of every turn, including discussion-only turns, update the conversation record, relevant subagent records, and the registry when necessary.
- Create one dedicated AI-record commit for that turn, containing only the relevant `AI_USAGE.md` and `ai-logs/` changes. Multiple writes during the turn belong to this single checkpoint. Do not create empty commits.
- Commit implementation changes and other project documentation separately. `AGENTS.md` changes are project-policy changes, not conversation-record changes.
- Use explicit owned file paths when staging. Inspect the staged diff before committing. If unrelated work is already staged, preserve it and report the blocked checkpoint rather than including, unstaging, or overwriting that work.
- Prepare the exact final reply before the checkpoint. Store an unsent draft as a separate `pending_final` object, with `transcript_status: "pending_final"`; keep confirmed `messages` limited to actual dialogue. Send the same reply after the checkpoint, or update the pending draft if the outcome changes.
- At the next opportunity, reconcile the pending reply against the actual conversation record, append the confirmed message, and remove `pending_final`. If delivery or source text cannot be verified, retain an explicit gap. Forced termination cannot be claimed to have produced a successful checkpoint.
- A no-commit restriction permits local recording only when writing is allowed. A no-write restriction permits no log writes. Report any remaining checkpoint gap; respect explicit no-push restrictions.
- Automatic pushing is authorized by default for this repository and does not require separate user confirmation. Explicit task restrictions, including no-push instructions, take precedence.
- In full subsession mode, the [full subsession protocol](docs/FULL_SUBSESSION.md) replaces automatic pushing by individual child sessions: only Main coordinates integration and remote pushes for the root task. Each child still records its turns and makes separate implementation and AI-record commits directly on `main` during an assigned serialized Git write window.
- Treat a fully handled user request, including a discussion-only turn, as a task completion boundary. Keep checkpoints local while that request is unfinished; do not wait for the entire project to be finished.
- At that boundary, automatically push the accumulated task-owned implementation, policy, and AI-record commits together in one push, after applicable checks pass, all task-owned changes are committed, the staging area is clear, and no unresolved conflicts or required user decisions remain. Conversation records must be current; the required `pending_final` draft does not block pushing. Do not push after each log edit or use scheduled jobs for this workflow.
- Immediately before pushing, fetch the destination and review the entire outgoing commit range, including earlier checkpoints for the same task. Unknown or unrelated outgoing commits block automatic pushing until their scope is resolved; preserve unrelated local changes.
- Use only existing `main` and its configured upstream. If missing, use `main` on an unambiguous existing remote; do not create another destination or push tags. Report ambiguous destinations instead of guessing.
- Never force-push or rewrite shared history. If the remote advances, reconcile its changes while preserving other contributors' work, repeat affected checks, and retry the normal push. If pushing remains blocked by access, branch protection, or network errors, preserve the local commits and report the actual failure; request input only when a user decision is needed.
- Report local recording, successful commits, and successful pushes as separate outcomes. Do not claim an action succeeded before checking its result.

## 4. Commit titles

- Use an Angular-style header: `<type>(<scope>): <summary>`.
- Every commit is title-only. Do not add a body or footer. This project adopts Angular's header conventions, not its mandatory-body rule.
- Use `build`, `ci`, `docs`, `feat`, `fix`, `perf`, `refactor`, or `test`; use `revert` for reversions.
- Always include a meaningful scope. Use `ai-logs` for AI-record checkpoints and `ai-policy` for these collaboration rules; use the affected project area for other changes.
- Write the summary in clear English, beginning with a lowercase imperative verb, without a trailing period. A short sentence is acceptable; do not force the title into a tiny phrase or turn it into a paragraph.
- Include a short conversation identifier and turn number in AI-record titles. These numbers are local to the conversation, not a shared repository counter.

Examples:

```text
docs(ai-policy): define AI disclosure and conversation logging rules
docs(ai-logs): record 1f6a turn 003 on the agent registry format
fix(parser): handle empty input before reading the first record
```

## 5. Collaboration across contributors and devices

- Read the existing registry and relevant session record before adding entries or resuming work. Do not overwrite other contributors' records or claim their work as your own.
- Reconcile the registry against the shared branch before integrating a new agent entry. Merge duplicate stable keys into one entry and combine verified model/tool lists.
- Concurrent registrations can allocate the same number. Resolve such conflicts before integration: renumber the entry not yet integrated and update its references together. Never renumber an already integrated agent entry; keep both ID and stable key in logs to detect mismatches.
- Distinct conversations use distinct UUIDs. Resuming the same conversation requires an explicit writer handoff and deduplication against its existing record. Sessions work on existing `main` with disjoint task-owned files; same-file edits and all staging, commits, and pushes are serialized through one Git writer.
- Resolve conflicts while preserving other contributors' changes. Do not rewrite shared history as a shortcut or alter global Git settings to implement these rules.
- Use project-relative paths and reusable rules. Keep personal greetings, personal background routes, machine-specific locations, and credentials outside the shared project policy.
- Preserve the repository's local-only ignore boundaries, including `process.md` and `runtime/`; do not include them in AI-record commits or publish them as logs.
- Archive task-owned chats confirmed finished, superseded, or no longer useful after preserving work, handoff, and dialogue records. Keep chats with pending work, required decisions, or recovery duties; idle alone does not mean useless.

## 6. Optional full subsession mode

- Enable only on an explicit `full subsession` or `full session` request for one root task; policy editing does not activate it.
- When entering the mode, starting a delegated role, or recovering lost policy context, read [docs/FULL_SUBSESSION.md](docs/FULL_SUBSESSION.md) and relevant durable task state. See [docs/FULL_SUBSESSION.zh-CN.md](docs/FULL_SUBSESSION.zh-CN.md) for Chinese review. Ordinary tasks need not load the full protocol.
- Starting prompts include the protocol path, shared `state_root`, role, task and parent/root IDs. All roles follow the protocol; all commits go directly to `main`; only Main coordinates serialized Git writes and remote pushes, with no independent child pushes.
