# Smart Commit

`smart-commit` 是一个面向 Codex 的 Git 提交 Skill。它会先分析本地变更，再按功能意图组织成可审查的原子提交，并在获得明确批准后执行提交。

## 功能

- 区分 staged、unstaged 和 untracked 变更，保护已有暂存边界。
- 检查疑似密钥、生成文件、缓存和大文件等风险项。
- 按独立功能或修复拆分原子提交，必要时支持 hunk 级分组。
- 生成符合 Conventional Commits 的中文或英文提交消息。
- 默认在提交前一次性展示分组、精确消息、文件范围、风险和检查结果，只需一次执行确认；也可使用 `--yes` 明确授权免确认提交。
- 不自动 push、stash、rebase 或 amend，也不会通过 `--yes` 绕过风险检查、验证或 Git hooks。

## 安装

个人 Skill 放在 `~/.codex/skills` 后，可在不同仓库中使用。详见 [OpenAI Codex Skills 说明](https://learn.chatgpt.com/use-cases/reusable-codex-skills)。

macOS / Linux：

```bash
git clone https://github.com/zade23/smart-commit.git ~/.codex/skills/smart-commit
```

Windows PowerShell：

```powershell
git clone https://github.com/zade23/smart-commit.git "$env:USERPROFILE\.codex\skills\smart-commit"
```

## 使用

在有本地变更的 Git 仓库中调用：

```text
$smart-commit
$smart-commit --lang zh
$smart-commit --lang en
$smart-commit --yes
$smart-commit --yes --lang zh
```

默认模式未指定语言时，Skill 会在生成整合执行预览前询问使用中文还是英文；`--yes` 模式不会询问，而是依次采用明确的自然语言要求、仓库近期风格和英文兜底。你也可以直接用自然语言要求它只分析或预览，不创建提交。

默认提交只需一次确认：批准包含变更分组、精确消息、文件范围、风险处理和检查结果的整合执行预览。

使用 `--yes` 时，本次调用会被视为执行授权。Skill 仍会完成分析、风险筛查、消息生成、检查和事务验证，但不会暂停询问，预览后直接提交。遇到歧义、冲突、未明确授权的敏感文件或检查失败时，会保持零提交并停止。
