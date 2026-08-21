# Commit conventions

Use this reference after logical groups are known. Repository-specific, explicit rules take precedence. Safety and the approval gate do not.

## Message shape

```text
<type>[optional scope][optional !]: <subject>

[optional body]

[optional footer(s)]
```

Keep `type`, `scope`, and machine-readable trailer tokens in English. Use the selected language for subject/body and explanatory footer values.

Fallback limits when the repository defines none:

- full header: at most 100 characters;
- English subject: at most 20 words;
- subject: one line, no terminal period or equivalent full stop;
- English subject: imperative present tense with a lowercase first word;
- Chinese subject: concise verb-object wording; avoid filler such as “进行” when a direct verb works.

Do not add emoji.

## Choose the type from the primary intent

| Type | Use when the group primarily... | Do not use merely because... |
| --- | --- | --- |
| `feat` | adds user- or API-visible capability | a new file was created |
| `fix` | corrects faulty observable behavior | code was edited after review |
| `docs` | changes only documentation | a feature includes supporting docs |
| `style` | changes formatting without semantic behavior | CSS or UI appearance changed |
| `refactor` | restructures code without adding behavior or fixing a defect | implementation moved while behavior also changed |
| `perf` | improves measured or clearly intended performance | code became shorter |
| `test` | adds or corrects standalone tests | tests accompany a feature or fix |
| `build` | changes build tooling, packaging, or dependency build behavior | application runtime configuration changed |
| `ci` | changes CI pipelines or automation | a local development script changed |
| `chore` | performs maintenance not represented above | the intent has not been investigated |
| `revert` | intentionally reverses an identified earlier commit | code happens to resemble an older version |

Use `chore` as a considered fallback, not a miscellaneous bucket.

## Scope

Use a scope only when it names a stable component meaningful to repository readers.

Scope preference:

1. explicit repository vocabulary or commitlint enum;
2. stable package/service/module name;
3. an established scope visible in recent history.

Use lowercase unless the repository requires otherwise. Omit scope for cross-cutting work or when the only candidate is a transient filename or directory fragment. Never use `*` unless repository rules explicitly adopt it.

## Subject, body, and footer

The subject says what changed, not that a change happened.

Use a body only when it adds durable information such as motivation, previous behavior, constraints, or a non-obvious implementation decision. Separate it from the header with one blank line. Do not repeat the diff or list filenames.

Use footers only for known metadata:

- Add `BREAKING CHANGE: <explanation>` or `!` only for an actual incompatible change. Keep `BREAKING CHANGE` in English; localize its explanation.
- Add issue trailers only when an issue identifier is supplied or reliably present in repository context. Never invent one.
- Follow repository-specific trailer syntax when defined.

## Atomic grouping examples

### Keep related artifacts together

These belong in one `feat(auth)` commit when they implement the same capability:

- authentication handler;
- its database migration;
- focused tests;
- configuration example;
- documentation needed to use the feature.

Do not split them into `feat`, `test`, and `docs` solely by file kind.

### Split independent fixes

An upload timeout fix and an unrelated navigation bug should normally be two `fix` commits even if both touch a shared utility file. Use verified hunk-level staging when safe.

## Bilingual examples

Good English:

```text
feat(history): add paginated event lookup

Return stable cursors so clients can resume long history queries without
reloading earlier pages.
```

```text
fix(auth): preserve session after token refresh
```

Good Chinese:

```text
feat(history): 添加事件分页查询

返回稳定游标，使客户端可以继续长历史查询而无需重复加载已有页面。
```

```text
fix(auth): 修复令牌刷新后会话丢失
```

Breaking change in Chinese mode:

```text
feat(api)!: 统一任务创建响应结构

BREAKING CHANGE: 任务创建接口不再返回旧版嵌套字段。
```

Avoid:

```text
feat: update files
fix: fixed bug.
docs: 更新了一些内容。
chore: miscellaneous changes
✨ feat(auth): Add login feature.
```

Problems include vague intent, past tense, terminal punctuation, meaningless grouping, capitalization inconsistent with the fallback rule, and unsupported emoji.
