---
name: smart-commit
description: Analyze local Git changes, flag risky files, group work into atomic Conventional Commits, generate Chinese or English messages, and create them after one consolidated approval or an explicit --yes authorization. Use when the user asks to plan, organize, or execute Git commits. Do not use for pushing, rebasing, amending, or rewriting history.
---

# Smart Commit

Create a reviewable commit plan, then execute it without losing, broadening, or silently rewriting the user's work.

## Invocation

Accept these explicit language forms:

```text
$smart-commit
$smart-commit --lang zh
$smart-commit --lang=zh
$smart-commit --lang en
$smart-commit --lang=en
$smart-commit --yes
$smart-commit --yes --lang zh
$smart-commit --lang=en --yes
```

Also accept an unambiguous natural-language request for Chinese or English. The formal options are `--lang zh|en` and `--yes`; when combined, they may appear in either order.

- Reject unknown options, missing values, repeated options, and conflicting flag/natural-language choices. Show the valid forms and do not mutate Git.
- `--yes` is explicit no-prompt execution authorization for the eligible batch produced by this invocation. It skips the consolidated approval question, not analysis, risk screening, checks, transactional verification, hooks, or the final audit.
- Without `--yes`, ask for Chinese or English before rendering the consolidated execution preview when the language is still unresolved. Language selection is not execution approval.
- With `--yes` and no language choice, do not ask. Prefer an unambiguous natural-language request, then stable recent repository style, then English as the fallback.
- Honor natural-language requests to analyze or preview without committing. Do not invent a `--dry-run` option.
- Reject `--yes` when combined with an analysis-only or preview-only request because the requested outcomes conflict.
- Do not invent aliases such as `-y`, `--force`, `--no-confirm`, or any option that bypasses hooks or safety checks.

## Non-negotiable safeguards

- Treat the user's initial request as permission to inspect and plan, not as execution authorization, unless the invocation includes a valid `--yes`.
- Do not stage, unstage, commit, or otherwise change the index before consolidated approval or valid `--yes` authorization.
- `--yes` authorizes only the exact eligible batch derived from the current invocation. It does not broaden the candidate set, include a hard-risk path without an explicit per-path instruction, accept failed checks, resolve ambiguity, or authorize any otherwise forbidden Git operation.
- Never use `git add -A`, `git add .`, `git commit -a`, `--no-verify`, `git reset --hard`, or any command that discards working-tree content.
- Never push, stash, amend, rebase, rewrite history, edit source files, or modify `.gitignore` as part of this skill.
- Keep unrelated user changes intact. Do not clean the worktree or include a file merely because it is modified.
- Stop on ambiguity that could include unapproved content or lose an existing staged selection.

## 1. Validate the repository and local rules

Perform a read-only preflight:

1. Confirm the current directory belongs to one Git worktree and identify its root, Git directory, branch, HEAD, and any superproject.
2. Inspect `git status` in a machine-readable form with all untracked files expanded rather than collapsed to directory names. Detect unmerged paths and merge, rebase, cherry-pick, or revert state. Stop on conflicts or an in-progress history operation. Warn and ask before proceeding from detached HEAD, a submodule, or a nested repository; in `--yes` mode, stop and report instead of asking.
3. Search from the working directory to the repository root for applicable `AGENTS.md`, `CONTRIBUTING*`, commitlint/commitizen configuration, commit templates, hook configuration, and documented validation commands.
4. Inspect a small sample of recent commit subjects only to infer stable scope names and local style. History is advisory, never stronger than explicit repository rules.
5. Explicit repository rules override the fallback message rules. If they conflict with the approval gate, content safety, or logical atomicity, stop and ask the user.

If the repository has no changes, report that and stop. Do not create an empty commit.

## 2. Select the candidate change set without mutation

Separate staged, unstaged, untracked, and ignored changes.

- If only staged changes exist, analyze only the staged diff.
- If no staged changes exist, analyze tracked working-tree changes and non-ignored untracked files.
- If staged and unstaged/untracked changes coexist, show the two sets separately. Unless the user clearly requested reorganizing all visible non-ignored changes, conservatively plan only the current staged selection and list the rest outside the batch. The user can revise this boundary from the consolidated preview. Preserve the current index until execution approval.
- Never force-add ignored files. Mention an ignored file only when the user explicitly asks about it, and keep it outside this workflow.

Record the selected source boundary. A staged-only selection must never absorb unstaged hunks from the same file later.

## 3. Screen risky candidates

Screen paths, file kinds, sizes, and relevant diff content before grouping. Prefer repository-provided secret scanners or pre-commit checks when available.

Hard-risk candidates include likely private keys, tokens, credential files, real `.env` files, password/secret files, service-account credentials, and machine-local sensitive configuration.

- Exclude each hard-risk path by default.
- Report only its path and risk category. Never echo a suspected secret or a matching line.
- Never include a hard-risk path in an executable preview without an explicit per-path instruction from the user. Otherwise exclude it by default and show that decision in the consolidated preview.
- When searching content, use redacted or file-name-only output rather than commands that print matches.

Soft-risk candidates include generated output, caches, logs, editor state, vendored dependency trees, minified/binary artifacts, and unexpectedly large files.

- Warn with the path, reason, and available size/type information.
- Propose whether to keep or exclude each item and show the decision in the consolidated preview so the user can approve or revise it there.
- In `--yes` mode, exclude a soft-risk item unless the invocation explicitly includes that path or repository rules clearly require it.

Apply screening to tracked files too; `.gitignore` does not protect content already tracked. Describe the result as heuristic, not proof that the changes contain no secrets.

## 4. Build a logically atomic commit plan

Read the selected diffs deeply enough to understand intent. Group by independent feature, fix, or maintenance purpose before assigning a type.

- Do not create broad buckets merely because files share `feat`, `fix`, or `docs`.
- Keep implementation with its directly related tests, migrations, configuration, and documentation. Use `test` or `docs` only for standalone test or documentation work.
- Split unrelated changes even when they touch the same subsystem.
- When one file contains multiple intents, propose hunk-level ownership. Show enough hunk context for the user to verify the split. If the split is not reliable, stop and ask the user to separate or choose an ownership rule.
- Order groups by dependency. Prefer commits that are independently understandable and, when practical, independently valid.
- Do not impose a maximum commit count.

Read [references/commit-conventions.md](references/commit-conventions.md) when assigning types/scopes or writing messages.

Carry a numbered grouping plan forward into the consolidated execution preview. For each group record:

- intent and rationale;
- proposed `type` and optional `scope`;
- complete paths, plus hunk descriptions where a file is split;
- dependency order;
- hard/soft-risk decisions and excluded paths;
- changes intentionally left outside the batch.

Do not request separate grouping approval. If ambiguity prevents a safe plan, ask a focused clarifying question; the answer resolves the plan but is not execution approval. In `--yes` mode, stop with zero commits and report the ambiguity instead of asking.

## 5. Render one complete preview and obtain execution authorization

After the grouping plan is internally complete:

1. Resolve the message language from the invocation. Without `--yes`, ask the user to choose Chinese or English when still unresolved. With `--yes`, apply its non-interactive language fallback.
2. Render the exact header, optional body, and optional footers for every group.
3. Discover relevant non-rewriting checks from repository instructions and project configuration. Run focused checks when practical. Run long checks only when the user already requested them or repository rules require them; otherwise list them as not run with the reason. Never run a formatter in write mode.
4. State honestly whether checks cover the complete worktree or an individual planned commit; do not claim isolated-commit validation when unstaged changes were present.
5. If a required or relevant check fails, show it prominently in the consolidated preview. In normal mode, execute only when the single approval explicitly authorizes proceeding despite that named failure; hooks may still reject the commit. In `--yes` mode, stop with zero commits and report the failure instead of asking or treating prior authorization as acceptance of an unknown failure.

Present one consolidated execution preview containing:

- every numbered group's intent, rationale, `type`, optional scope, exact message, complete file/hunk membership, and dependency order;
- the selected source boundary;
- hard/soft-risk decisions, excluded paths, and changes left outside the batch;
- checks run, results, and coverage limitations;
- an explicit authorization statement: in normal mode, approval authorizes staging and committing the batch exactly as shown; in `--yes` mode, the invocation has already authorized that exact batch.

In normal mode, ask for one unambiguous approval to execute the entire batch. This is the workflow's only approval gate; do not ask for separate grouping and execution approvals. A revision invalidates the preview, so rebuild it and ask once for approval of the revised complete preview.

In `--yes` mode, present the same complete preview as a non-blocking progress update, then continue directly to execution without asking or pausing. The flag authorizes only that exact preview. If the safe preview cannot be completed, return a zero-commit summary rather than seeking input.

For analysis-only requests, stop after the preview without asking for execution approval and return a zero-commit summary.

## 6. Execute the authorized batch transactionally

Only after consolidated execution approval or a valid `--yes` authorization:

1. Capture the baseline HEAD (including an unborn-branch case), machine-readable status, staged path list, and binary staged diff in a temporary location outside the repository. Do not expose diff contents in the summary.
2. Prepare only the first authorized group in the index using explicit paths or verified patch staging. Preserve the working tree.
   - For staged-only processing, construct groups only from the original cached diff.
   - If a path has both staged and unstaged hunks, never use whole-file `git add` in staged-only mode.
   - Use index-only unstage operations compatible with the repository's HEAD state. If safe restoration cannot be guaranteed, stop before changing the index.
3. Compare the complete staged diff with the authorized group, not just its path list. Stop on any extra, missing, or altered hunk.
4. Write the authorized message to a temporary file outside the repository and commit with that file so multiline messages and characters are preserved. Honor signing configuration and all hooks.
5. Record the new SHA and verify the created commit's diff and message against the authorized preview. Re-read status and detect hook-created or otherwise unexpected changes.
6. Continue with the next group only when the previous commit and current repository state match expectations.

If a failure happens before any commit succeeds, restore the original index from the captured baseline when it is safe and verifiable. Never change the working tree to perform restoration.

If one or more commits succeeded, do not automatically roll them back, amend them, or reset HEAD. Stop at the first failure, preserve remaining changes, and report the partial result. If a hook or external process changed files or the index unexpectedly, stop without attempting a speculative repair.

## 7. Return an auditable summary

For every outcome, report:

- whether the result is success, partial success, or zero commits;
- whether execution used consolidated approval or `--yes` authorization;
- each created SHA and exact subject, with its paths/hunks;
- checks run and their results;
- excluded risky files and the reason category, without sensitive values;
- planned groups not committed and why;
- remaining staged, unstaged, and untracked paths;
- confirmation that no push occurred.

Keep temporary message and baseline artifacts outside the repository and remove them when they are no longer needed and doing so is safe.
