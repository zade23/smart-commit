# Smart Commit

`smart-commit` 是一个面向 Codex 的 Git 提交 Skill。它会先分析本地变更，再按功能意图组织成可审查的原子提交，并在获得明确批准后执行提交。

## 功能

- 区分 staged、unstaged 和 untracked 变更，保护已有暂存边界。
- 检查疑似密钥、生成文件、缓存和大文件等风险项。
- 按独立功能或修复拆分原子提交，必要时支持 hunk 级分组。
- 生成符合 Conventional Commits 的中文或英文提交消息。
- 提交前提供分组确认和最终确认，不自动 push、stash、rebase 或 amend。

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
```

不指定语言时，Skill 会在分组方案确认后询问使用中文还是英文。你也可以直接用自然语言要求它只分析或预览，不创建提交。

提交始终需要两次确认：先批准变更分组，再批准包含精确消息和文件范围的最终预览。
