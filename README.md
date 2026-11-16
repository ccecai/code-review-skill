# Code Review Skill

[English](README.md) | [简体中文](README.zh-CN.md)

A reusable code review workflow for Codex, Cursor, and eligible ChatGPT accounts.
The skill instructions are written in English; review results use plain Chinese
by default. It checks whether changes are ready to merge, with particular attention
to module boundaries, failure handling, concurrency, and embedded resource costs.

## Quick start

Install the skill using one of the methods below, then ask:

```text
代码审查
```

or:

```text
code review
```

These phrases request a review; merely quoting them or editing a review prompt
does not start one. Automatic selection depends on the host discovering the skill
and judging it relevant. In Codex, explicitly invoke `$code-review` when needed.
In Cursor, select `/code-review` from the chat command picker.

## What it reviews

1. Correct behavior and whether a bug fix addresses the shared root cause.
2. Module responsibilities, layering, interfaces, and business coupling.
3. Necessary changes, existing functions, and unnecessary dependencies or refactoring.
4. Names, comments, readability, and maintenance.
5. Existing features, target-branch interactions, protocols, and stored-data compatibility.
6. State transitions, event order, repeated calls, timers, and callbacks.
7. Errors, timeouts, cleanup, bounded retries, and recovery.
8. Input validation, data bounds, conversions, units, and security where relevant.
9. Races, locks, object lifetime, interrupt contexts, and peripheral conflicts.
10. RAM, Flash, heap, stack, CPU loading, latency, power, and hardware wear.
11. Reproduction, regression checks, builds, and hardware verification evidence.

Hardware checks apply when relevant. Resource savings must preserve correctness,
safety, timing, and reliability. The reviewer must distinguish measurements from
estimates and must not invent performance figures.

## Install

The commands below use a POSIX shell, such as macOS Terminal or a Linux shell,
and assume Git is installed. For a private repository, authenticate with a GitHub
account that has access. Do not put an access token in the URL or command history.
If the destination already exists, inspect and back it up before replacing it;
the clone commands intentionally do not overwrite an existing installation.

### Codex: personal installation

For the default Codex home:

```sh
mkdir -p "$HOME/.codex/skills"
git clone https://github.com/ccecai/code-review-skill.git "$HOME/.codex/skills/code-review"
```

If you use a custom `CODEX_HOME`, install into its `skills/code-review` directory
instead. Start a new session or restart the client if the skill is not discovered.

### Cursor: personal installation

```sh
mkdir -p "$HOME/.cursor/skills"
git clone https://github.com/ccecai/code-review-skill.git "$HOME/.cursor/skills/code-review"
```

Reopen Cursor if needed. These are local user-level skills; remote and cloud
environments may need separate installation or supported skill syncing.
See the [Cursor skills documentation](https://cursor.com/docs/skills).

For a repository-specific installation, place the skill in
`.cursor/skills/code-review/` or `.agents/skills/code-review/` inside that project.
The folder containing `SKILL.md` should be named `code-review`.

### ChatGPT: import into an eligible account

1. Download [code-review.zip](code-review.zip). When viewing the file on GitHub,
   use its download control to save the actual ZIP.
2. In ChatGPT, open **Plugins → Skills → Create → Upload from your computer**.
3. Upload the ZIP and complete the product's scan and installation flow.
4. Start a chat and request a review. Provide code, a diff, files, or an accessible
   repository through the tools available in that chat.

ChatGPT skills depend on account eligibility and workspace settings. A local Codex
or Cursor installation does not install the skill into ChatGPT. If ChatGPT says
the account cannot use skills, this ZIP cannot enable that feature.
See [Skills in ChatGPT](https://help.openai.com/en/articles/20001066-skills-in-chatgpt).

## Usage examples

Review all uncommitted changes:

```text
代码审查：检查当前暂存、未暂存以及相关的未跟踪文件。
```

Review a branch for a specific sprint:

```text
code review: review this branch against origin/sprint-2026-11.
Focus on concurrency, compatibility, RAM/Flash growth, and CPU loading.
Report the findings in Chinese.
```

Replace the example branch with the actual target. The skill must identify the
base and report when target-branch information may be stale; it must not guess
what “latest sprint” means.

Request explicit invocation in Codex:

```text
$code-review Review the current changes and explain the findings in plain Chinese.
```

## Review behavior and report

Without a specified scope, the workflow inspects staged, unstaged, and relevant
untracked files. A requested branch, PR, commit, or file takes precedence. It reads
related callers, callees, shared state, configuration, and tests beyond the diff.

Findings come first, ordered by severity:

- **P0:** Critical safety/security problems or widespread failures needing immediate action.
- **P1:** Problems that must be fixed before merging.
- **P2:** Ordinary defects that should be fixed.
- **P3:** Low-impact defects.

Each finding includes a verified location, trigger, concrete consequence,
supporting evidence, and the smallest practical fix. Confirmed defects, risks
needing validation, and optional improvements are separated. The report finishes
with the scope, base when relevant, checks performed, checks not performed, and
whether any findings block merging.

Illustrative report format, not an actual repository finding:

```text
[P1] 超时后未释放锁，后续通信会一直等待
位置：src/example.c:42（示例位置）
触发：通信等待超时，函数提前返回。
后果：下一次调用无法拿到锁，通信无法继续。
依据：提前返回分支跳过了公共释放步骤。
建议：让失败分支也经过现有的资源释放逻辑。

验证：仅静态检查；未运行构建或实机测试。
合入：修复此问题后再合入。
```

## Boundaries

- Reviews do not edit code unless fixes are explicitly requested.
- Reviews do not commit, stash, reset, rebase, or switch branches.
- Applicable `AGENTS.md` and project rules remain in effect.
- The skill adds instructions, not repository access, shell tools, or permissions.
- Missing build, test, device, or target-branch evidence must be disclosed.
- “No findings” does not mean the code is proven defect-free.

## Files and updates

```text
code-review-skill/
├── SKILL.md              # English review instructions and trigger description
├── agents/
│   └── openai.yaml       # OpenAI UI metadata and implicit invocation policy
├── README.md             # English introduction and usage
├── README.zh-CN.md       # Chinese introduction and usage
└── code-review.zip       # Import package containing only the skill files
```

For a clean installation made with `git clone`, update from its installation
directory with `git pull --ff-only`. If local edits exist or Git refuses the
update, inspect the changes instead of resetting them. After editing `SKILL.md`
or `agents/openai.yaml`, rebuild `code-review.zip` with those two files under a
top-level `code-review/` folder. Reimport the updated package into ChatGPT through
the product's supported flow.

## References

The documentation layout was informed by public skill repositories, including
[Anthropic Skills](https://github.com/anthropics/skills) and
[OpenAI Skills](https://github.com/openai/skills). The review checklist comes from
the author's requirements. This is an independent skill, not an official OpenAI,
Anthropic, or Cursor product.

Additional reference: [Agent Skills specification](https://agentskills.io/specification).
