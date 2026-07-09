---
name: quick-commit
description: >
  Intelligent git commit assistant. Uses a one-shot Python script to gather
  all git state (status, filtered diffs, stats) in a single call, filtering
  meaningless content (binary, base64, lock files, build output). Generates
  conventional commit messages from actual diff content. Supports selective
  staging — only stages meaningful files unless user says "all"/"全部"/"全部提交".
  Also supports amending last commit message. Trigger when user says: commit,
  提交, 快速提交, 帮我commit, git commit, amend commit, 修改commit说明,
  修改提交, or any request to create or modify a git commit.
agent_created: true
---

# Quick Commit

Intelligent commit with one-shot data collection and content filtering.
No more `git commit -m "update"` or `git commit -m "1"`.

## Core Script

The script at `scripts/git-info.py` collects ALL git state in one call:

```
python <skill-dir>/scripts/git-info.py [--amend] [--max-diff N] [--max-total N]
```

It returns JSON with:
- `ok`: false if working tree is clean
- `branch`, `ahead`, `behind`: branch info
- `staged`, `unstaged`, `untracked`: file lists by status
- `files[]`: array of file entries, each with `file`, `status`, `category`, `diff` (filtered), `truncated`, `skipped`, `skip_reason`
- `noise_files`, `binary_files`: files auto-skipped
- `suggested_stage`, `suggested_skip`: which files to git add vs ignore
- `summary`: counts of total/meaningful/noise/binary/staged/unstaged

### What it filters

- **Noise files**: `package-lock.json`, `yarn.lock`, `bun.lock`, `.env`, `node_modules/`, `.workbuddy/`, `.vscode/`, `dist/`, `build/`
- **Binary files**: images, fonts, archives, executables, databases (by extension)
- **Content filtering**: base64 blobs (truncated to 80 chars), lines >300 chars (truncated), entire diffs >300 lines or >8000 chars (truncated)

## Workflow

### Step 1: Gather info

Run the ONE script. Do NOT run separate `git status` / `git diff` / `git log` commands.

**Determine the right command based on context:**

| User said | Command |
|-----------|---------|
| Default / no specific files mentioned | `python <skill-dir>/scripts/git-info.py --staged` — prioritizes staging area |
| "全部提交" / "commit all" / "全部" | `python <skill-dir>/scripts/git-info.py` — all changes |
| "针对当前修改文件" / "提交这些文件" / mentioned specific files in conversation | `python <skill-dir>/scripts/git-info.py --files "a.vue,b.vue"` |

**How to determine the file list for `--files`:**
Scan the conversation context for files that were read/written/edited in the current session. These are the "当前修改文件". Use their full paths relative to repo root, comma-separated.

For amend mode, add `--amend`:
```
python <skill-dir>/scripts/git-info.py --amend --files "a.vue,b.vue"
```

**CRITICAL**: `--staged` mode respects the user's staging intent. Noise files (`.workbuddy/`, `pnpm-lock.yaml`, etc.) that the user explicitly staged WILL get diffs and be included in `suggested_stage`. Unstaged noise files are still skipped.

Read the JSON output. If `ok` is false, tell the user the tree is clean and stop.

### Step 2: Stage changes

**In `--staged` or `--files` mode** (default): the script already targets the right files. Skip staging — just use the files in `suggested_stage` directly for the commit.

**In "全部提交" mode**: `git add -A`.

**In other modes** with unstaged source files: Only stage `suggested_stage` files: `git add <file1> <file2> ...`.

MUST always execute `git add` before committing EXCEPT in `--staged`/`--files` mode where the target files are already staged. `git add` is idempotent and harmless.

### Step 3: Generate commit message

Analyze the `diff` fields from `files[]` where `skipped` is false.
Only include files that have actual diff content.

**Format**: `<type>: <Chinese summary>`

Types:
- `feat`: New feature or functionality
- `fix`: Bug fix
- `refactor`: Code restructuring without behavior change
- `docs`: Documentation changes
- `style`: Formatting only (no code change)
- `chore`: Build process, tooling, dependencies
- `perf`: Performance improvement
- `test`: Adding or fixing tests

**Summary**: Concise Chinese (the user speaks Chinese), imperative mood, under 50 chars.

**Body**: If multiple logical change groups, append 2-5 bullet points in Chinese.
Each line under 72 chars.

### Step 4: Confirm and execute

Present the message and ask: "用这个 commit 说明？"

On confirmation (or if user said "直接"/"直接发布"): execute `git commit -m "<message>"`.

If user said "直接发布" or "push", also run `git push` after commit.

### Step 5: Report

Show `git log -1 --oneline`.

## Amend Mode

Triggered by: "amend", "修改commit", "修改提交说明", "修改commit说明"

1. Run `python <skill-dir>/scripts/git-info.py --amend` to get combined diff.
2. If there are new uncommitted changes, stage them first (following Step 2 rules).
3. Generate message from the combined diff.
4. Use `git commit --amend -m "..."`.
5. **Warn if pushed**: check `ahead` field. If ahead=0 and there's a remote tracking branch, warn that `--amend` + `--force` rewrites history.

## Edge Cases

- **Clean tree**: Script returns `ok: false`. Say "工作区没有变更，无需提交。"
- **Only noise files**: Script returns `meaningful_changed: 0`. In `--staged` mode, staged noise files ARE included (user's intent respected). In normal mode, say "所有变更都是自动生成/二进制文件，建议跳过。强制提交？"
- **Truncated diff**: File entry has `truncated: true`. The commit message should still be accurate based on the visible portion.
- **No branch** (detached HEAD): Generate message but warn user about detached state.
- **Merge conflict**: Don't proceed. Tell user to resolve conflicts first.
