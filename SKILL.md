---
name: project-standards
description: "Install or sync Brian's agent instruction standards in a repository — PROJECT_WORKFLOW.md, AGENTS.md with the multi-agent delegation policy, and CLAUDE.md importing it. Use when starting a new project, adopting the standards into an existing repo, or syncing many repos at once."
---

# Project standards

Three files define the standard, and their relationship is the whole point:

- **`PROJECT_WORKFLOW.md`** — the delivery workflow. Byte-identical across repos except its Project Configuration section, which is per-project.
- **`AGENTS.md`** — canonical shared agent instructions. Carries the multi-agent delegation policy and one managed workflow-reference block.
- **`CLAUDE.md`** — imports `AGENTS.md` with a bare `@AGENTS.md` line, then adds Claude-specific content. Never duplicates the policy.

A markdown link to `AGENTS.md` is **not** an import — Claude Code does not pull the file in. The first line must be `@AGENTS.md`.

## Resolve the assets first

In priority order, for each of `PROJECT_WORKFLOW.md`, `AGENTS.template.md`, `CLAUDE.template.md`, `multi-agent-policy.md`, `managed-block.md`, `config-detection.md`:

1. `~/project-standards/` and `~/project-standards/assets/`
2. Any already-governed repo on the machine (an `AGENTS.md` containing `## Mandatory Multi-Agent Coding Workflow`)
3. Ask Brian for the canonical `PROJECT_WORKFLOW.md`

Never reconstruct the workflow or the policy from memory. If you cannot obtain them, stop and say so.

## Pick the mode

- **Bootstrap** — a repo with none of the three files. Most of the work is configuration.
- **Adopt** — a repo with some of them, or with its own workflow document. Preservation is the work.
- **Fleet** — a directory containing many repos. Delegate to the utility; do not hand-roll it.

---

## Mode A — Bootstrap (greenfield)

1. Copy canonical `PROJECT_WORKFLOW.md` to the repo root, unmodified.
2. **Fill the Project Configuration** (see below). This is the step that makes the file useful rather than decorative.
3. Write `AGENTS.md` from `AGENTS.template.md`, appending anything project-specific you already know (stack, architecture, conventions) *below* the managed block.
4. Write `CLAUDE.md` from `CLAUDE.template.md`.
5. Create `CHANGELOG.md` if absent — the workflow's release section expects it.
6. Verify, then report.

Do not commit. Show what was created and let Brian commit.

## Mode B — Adopt (brownfield, one repo)

Order matters: the workflow file must exist and verify before any agent file references it.

1. **Audit before writing.** Record which of the three exist, whether `AGENTS.md` already has the policy, whether `CLAUDE.md` has the `@AGENTS.md` import, and whether either already references a workflow.
2. **An existing workflow document is not automatically replaceable.** If the repo has `WORKFLOW.md` or `PROJECT_WORKFLOW.md` that differs from canonical, show a diff and ask. It is very likely their own document with real content. Only its Project Configuration values may be carried into canonical automatically; anything outside that section differing means a manual merge decision.
3. **Never write to a file with uncommitted changes** unless Brian approves that exact file after seeing its diff. If `git status` *fails* — stale `index.lock`, `safe.directory`, timeout — treat the whole repo as off limits. A failed status read is not a clean tree.
4. Back up every file you modify, outside the repository, before modifying it.
5. Add the policy to `AGENTS.md` if missing; add the `@AGENTS.md` import to `CLAUDE.md` if missing. Preserve everything else — append rather than restructure.
6. Insert the managed block exactly once per agent file. If a block already exists, replace that block only. Rewrite nothing outside the markers.
7. **If an agent file already has a hand-written workflow reference**, do not add a second one silently. Report it and ask whether to normalize.

## Mode C — Fleet (many repos)

Use `sync-project-workflow.mjs` from `~/project-standards`. It already handles discovery, classification, backups, atomic writes, case-only renames on APFS, and resumable runs. Do not reimplement it.

```bash
node sync-project-workflow.mjs --dry-run --root <dir> --source <canonical> \
  --backup <dir>/backups --manifest <dir>/approval-manifest.json \
  --repos-cache repos.json --plan-cache plan.jsonl --time-budget 100
```

Dry-run → present the audit → get explicit approval in the manifest → `--apply` → `--check`. Nothing is approved by default and that is deliberate. On a slow or network filesystem the run is chunked: re-run the same command until it exits 0.

---

## Filling the Project Configuration

Read `config-detection.md` for the per-key mapping. In short: derive what the repo states, ask about the rest, invent nothing.

Derivable in practice (measured on a real Next.js/Supabase repo: 16 of 17 probed keys): repository name, remote, default branch, package manager, runtime, install/dev/test/lint/typecheck/build commands, CI check names, version source, changelog path, migration command, environment variable *names*.

Always ask: staging and production URLs and environment names, approval owner, manual production trigger, monitoring, observation period, rollback procedure, artifact registry, required reviews, merge method.

Three rules that matter more than coverage:

- **Verify a command exists before writing it in.** A script name that does not run is worse than a placeholder.
- **A value you cannot derive and Brian cannot answer stays a `{placeholder}`.** An honest gap beats a plausible fiction — especially in the deployment section, where a wrong value points a production deploy at the wrong target.
- **For greenfield, `not yet configured` is a legitimate answer.** Note in the report which gates are unenforceable until the infrastructure exists. A workflow claiming a staging gate the project does not have is worse than one admitting the gap.

Batch every question into one round grouped by section, showing your derived answer as an acceptable default. A dozen questions once beats sixty in sequence.

## Keeping prose honest

When you install or replace a workflow document, existing prose may now describe something else. After any change, check `AGENTS.md` and `CLAUDE.md` for:

- **Broken links** — replacing `WORKFLOW.md` with `PROJECT_WORKFLOW.md` breaks every `[WORKFLOW.md](WORKFLOW.md)` link. Repair bare references, but **never rewrite a path-qualified one** such as `docs/process/WORKFLOW.md`: that names a different file which probably exists.
- **False attributions** — sentences claiming the workflow "specifies" or "lists" something it does not. The canonical template leaves commands and paths as placeholders and never mentions Conventional Commits.
- **Stale descriptions** — a summary written for an older document that omits versioning, staging, manual production promotion and rollback.

## Verify before reporting

Every touched repo must satisfy all of:

1. Root `PROJECT_WORKFLOW.md` with exact canonical casing (check with `readdir`, not `existsSync` — on macOS `existsSync` answers yes for `Project_Workflow.md`).
2. Content canonical, or canonical body with only Project Configuration values differing.
3. Root `AGENTS.md` with exactly one policy section and exactly one managed block.
4. Root `CLAUDE.md` with `@AGENTS.md` as its first line and exactly one managed block, and no copy of the policy.
5. No unresolved `{CORRECT_RELATIVE_PATH}` anywhere.
6. Every workflow reference resolves on disk.
7. No circular import — `AGENTS.md` must not import `CLAUDE.md`.

`~/project-standards/audit-consistency.mjs` checks all of this across a machine.

## Hard rules

- **Never commit, stage, stash, reset, clean, branch or push.** Report what changed and let Brian commit.
- **Never modify anything outside** `PROJECT_WORKFLOW.md`, `AGENTS.md`, `CLAUDE.md` and the automation directory.
- **Back up before overwriting**, outside the repository, never overwriting a previous backup.
- **Skip and report** rather than guess: collisions, symlinks, ambiguous references, unreadable git state.
- Some environments forbid `unlink` while permitting `rename`. If a delete is refused, move the file to a quarantine directory instead of failing a completed install.

## Report

Files created, files modified, configuration values derived versus asked, values left as placeholders and why, anything skipped with the reason and the recommended next action, and where the backups are.
