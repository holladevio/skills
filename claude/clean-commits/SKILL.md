---
name: clean-commits
description: Commits the working tree as a series of logical commits with Conventional Commits messages, and offers to add the commit-msg, pre-commit and pre-push hooks when the repo lacks them. Use whenever the user asks to commit or save work to git, including short requests like "commit", "коммит", "сделай коммит", "закоммить" or "закомить".
---

# Clean commits

The user doesn't write commit messages by hand; they ask the agent to commit
and expect clean history in return. Commit messages follow Conventional
Commits and are checked by the `commit-msg` hook when the repo has one.

## 1. Look at what changed

Run `git status`, `git diff`, `git diff --cached` and `git log --oneline -10`.

Files the user already staged are their own grouping: commit them first, as
one commit, before grouping the rest. But look at the staged diff anyway:
if something there looks unexpected
(unrelated to the work, or a side effect of a tool), mention it in your
report.

Don't commit secrets (`.env`, keys, tokens) or junk (`.DS_Store`, build
output, logs), even when they show up as untracked. Leave them out, say so in
the report and suggest adding them to `.gitignore`.

## 2. Check the git hooks

Run `git config core.hooksPath` and look for the `commit-msg`, `pre-commit`
and `pre-push` hooks: in that directory, in the directory above it (Vite+
and husky point `core.hooksPath` at a `_` dispatcher folder and keep the
real hooks one level up), or in `.git/hooks` (the `*.sample` files there
don't count).

If any is missing and the user hasn't declined in this conversation, read
`references/setup-hooks.md` from this skill's directory and follow it
before the work commits.

## 3. Split into logical commits

One commit = one reason to change. A feature, a bug fix and an unrelated
docs tweak are three commits, even if they were made in one session. Group by
intent, not by file or by package: a change to a shared package together
with the app code that adapts to it is one commit.

Stage each group by path (`git add <paths>`). Interactive staging
(`git add -p`) is not available; if one file mixes unrelated changes and
can't be split by path, keep it in the commit it mostly belongs to and say
so in the report.

## 4. Write the message

Read the `commit-msg` hook for the allowed types and scopes. It is the
single source of truth, so don't rely on memory: the list changes as
packages are added. Without a hook, use the types from `assets/commit-msg`
and one scope per workspace package.

```
<type>(<scope>): <subject>

<body>
```

- **type**: `feat` adds a capability, `fix` corrects behavior, `refactor`
  changes code without changing behavior, `style` changes how code looks
  (formatter and lint rules together with the code they reformat), `build`
  covers dependencies and the build toolchain, `chore` covers other
  maintenance.
- **scope**: the workspace package the change is about. Omit it when the
  change spans the repo (root configs, several packages equally).
- **subject**: imperative, lowercase, no trailing period. It says what the
  commit does: `add users route`, not `added users route` or `users route`.
  The whole first line, type and scope included, is at most 72 chars.
- **body**: explain why, wrapped at 72. Skip it when the subject already
  says everything.
- **breaking change**: add `!` before the colon (`feat(shared)!:`,
  `feat!:`) and a `BREAKING CHANGE:`
  footer describing what callers must change. Renaming or removing a field
  in a shared package or an API route is breaking.

Keep any trailers the environment asks for (e.g. `Co-Authored-By`) as the
last lines of the message.

Commit with a heredoc so the formatting survives:

```bash
git commit -F - <<'EOF'
feat(server): add users route

Body.
EOF
```

## 5. When a hook rejects the commit

Hooks run on every commit. A failure means something is wrong, so fix the
cause and commit again:

- `commit-msg` failed: rewrite the message.
- `pre-commit` failed: fix the lint or type error, re-stage, commit again.
  If the hook autofixed files, check that the fixes are staged.
- `pre-push` failed (only when the user asked to push): fix the cause in a
  new commit and push again.

If the failure is in code this work didn't touch, don't fix it on the side:
show the user the output and ask how to proceed.

Never use `--no-verify` and don't amend earlier commits to work around a
failure.

## 6. Report

List the commits you created (short hash and subject) and anything you left
uncommitted, with the reason. Don't push unless the user asks.
