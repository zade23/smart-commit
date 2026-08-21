---
name: smart-commit
description: Analyze local Git changes, flag risky files, group work into atomic Conventional Commits, generate Chinese or English messages, and create only the commits the user explicitly approves. Use when the user asks to plan, organize, or execute Git commits. Do not use for pushing, rebasing, amending, or rewriting history.
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
```

Also accept an unambiguous natural-language request for Chinese or English. The only formal v1 option is `--lang zh|en`.

- Reject unknown options, missing values, repeated language options, and conflicting flag/natural-language choices. Show the valid forms and do not mutate Git.
- Without a language choice, wait until the grouping plan is approved, then ask Chinese or English.
- Honor natural-language requests to analyze or preview without committing. Do not invent a `--dry-run` option.
- Do not accept an option that bypasses either approval gate.

## Non-negotiable safeguards

- Treat the user's initial request as permission to inspect and plan, not as final approval to stage or commit.
- Do not stage, unstage, commit, or otherwise change the index before the final execution approval.
- Never use `git add -A`, `git add .`, `git commit -a`, `--no-verify`, `git reset --hard`, or any command that discards working-tree content.
- Never push, stash, amend, rebase, rewrite history, edit source files, or modify `.gitignore` as part of this skill.
- Keep unrelated user changes intact. Do not clean the worktree or include a file merely because it is modified.
- Stop on ambiguity that could include unapproved content or lose an existing staged selection.

## 1. Validate the repository and local rules

Perform a read-only preflight:

1. Confirm the current directory belongs to one Git worktree and identify its root, Git directory, branch, HEAD, and any superproject.
2. Inspect `git status` in a machine-readable form with all untracked files expanded rather than collapsed to directory names. Detect unmerged paths and merge, rebase, cherry-pick, or revert state. Stop on conflicts or an in-progress history operation. Warn and ask before proceeding from detached HEAD, a submodule, or a nested repository.
3. Search from the working directory to the repository root for applicable `AGENTS.md`, `CONTRIBUTING*`, commitlint/commitizen configuration, commit templates, hook configuration, and documented validation commands.
4. Inspect a small sample of recent commit subjects only to infer stable scope names and local style. History is advisory, never stronger than explicit repository rules.
5. Explicit repository rules override the fallback message rules. If they conflict with the approval gates, content safety, or logical atomicity, stop and ask the user.

If the repository has no changes, report that and stop. Do not create an empty commit.

## 2. Select the candidate change set without mutation

Separate staged, unstaged, untracked, and ignored changes.

- If only staged changes exist, analyze only the staged diff.
- If no staged changes exist, analyze tracked working-tree changes and non-ignored untracked files.
- If staged and unstaged/untracked changes coexist, show the two sets separately and ask whether to process only the current staged selection or reorganize all visible non-ignored changes. Preserve the current index until final approval.
- Never force-add ignored files. Mention an ignored file only when the user explicitly asks about it, and keep it outside this workflow.

Record the selected source boundary. A staged-only selection must never absorb unstaged hunks from the same file later.

## 3. Screen risky candidates

Screen paths, file kinds, sizes, and relevant diff content before grouping. Prefer repository-provided secret scanners or pre-commit checks when available.

Hard-risk candidates include likely private keys, tokens, credential files, real `.env` files, password/secret files, service-account credentials, and machine-local sensitive configuration.

- Exclude each hard-risk path by default.
- Report only its path and risk category. Never echo a suspected secret or a matching line.
- Require an explicit per-path decision before including it in the later plan.
- When searching content, use redacted or file-name-only output rather than commands that print matches.

Soft-risk candidates include generated output, caches, logs, editor state, vendored dependency trees, minified/binary artifacts, and unexpectedly large files.

- Warn with the path, reason, and available size/type information.
- Let the user keep or exclude each item during the grouping review.

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

Present the first approval gate as a numbered plan containing:

- intent and rationale;
- proposed `type` and optional `scope`;
- complete paths, plus hunk descriptions where a file is split;
- dependency order;
- hard/soft-risk decisions and excluded paths;
- changes intentionally left outside the batch.

Ask the user to approve or revise the grouping. This approval is still non-mutating.

## 5. Choose language, render messages, and check the work

After grouping approval:

1. Resolve the message language from the invocation or ask the user to choose Chinese or English.
2. Render the exact header, optional body, and optional footers for every group.
3. Discover relevant fast, non-rewriting checks from repository instructions and project configuration. Run focused checks when practical. Explain and ask before long-running checks. Never run a formatter in write mode.
4. State honestly whether checks cover the complete worktree or an individual planned commit; do not claim isolated-commit validation when unstaged changes were present.
5. If a required or relevant check fails, show the failure and pause. Proceed only after the user explicitly decides how to handle it; hooks may still reject the commit.

Present a final batch preview with exact messages, exact file/hunk membership, check results, exclusions, and order. Ask for one unambiguous approval to execute the entire batch. Any revision invalidates the preview and requires a fresh final approval.

For analysis-only requests, stop after the preview and return a zero-commit summary.

## 6. Execute the approved batch transactionally

Only after final approval:

1. Capture the baseline HEAD (including an unborn-branch case), machine-readable status, staged path list, and binary staged diff in a temporary location outside the repository. Do not expose diff contents in the summary.
2. Prepare only the first approved group in the index using explicit paths or verified patch staging. Preserve the working tree.
   - For staged-only processing, construct groups only from the original cached diff.
   - If a path has both staged and unstaged hunks, never use whole-file `git add` in staged-only mode.
   - Use index-only unstage operations compatible with the repository's HEAD state. If safe restoration cannot be guaranteed, stop before changing the index.
3. Compare the complete staged diff with the approved group, not just its path list. Stop on any extra, missing, or altered hunk.
4. Write the approved message to a temporary file outside the repository and commit with that file so multiline messages and characters are preserved. Honor signing configuration and all hooks.
5. Record the new SHA and verify the created commit's diff and message against the approved preview. Re-read status and detect hook-created or otherwise unexpected changes.
6. Continue with the next group only when the previous commit and current repository state match expectations.

If a failure happens before any commit succeeds, restore the original index from the captured baseline when it is safe and verifiable. Never change the working tree to perform restoration.

If one or more commits succeeded, do not automatically roll them back, amend them, or reset HEAD. Stop at the first failure, preserve remaining changes, and report the partial result. If a hook or external process changed files or the index unexpectedly, stop without attempting a speculative repair.

## 7. Return an auditable summary

For every outcome, report:

- whether the result is success, partial success, or zero commits;
- each created SHA and exact subject, with its paths/hunks;
- checks run and their results;
- excluded risky files and the reason category, without sensitive values;
- planned groups not committed and why;
- remaining staged, unstaged, and untracked paths;
- confirmation that no push occurred.

Keep temporary message and baseline artifacts outside the repository and remove them when they are no longer needed and doing so is safe.
