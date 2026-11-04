# Set up the git hooks

The user starts new projects often and won't remember to add hooks, so the
skill checks for them on every commit and this is how a new repo gets
them.

Tell the user which hooks are missing and what each would run, and ask
whether to add them. If they decline, go on with the commit and don't ask
again in this conversation. The setup changes the project, so its files go
into their own commit (`build: add git hooks`) before the work commits. If
a file the setup touches, such as `package.json`, already has uncommitted
work changes, don't mix them in: commit the hooks after the work instead.

## The hooks

Hooks live in `.hooks/`:

- **commit-msg** checks the message. Copy `assets/commit-msg` from this
  skill's directory and keep it executable. Fill `scopes` with one short
  name per workspace package (`@acme/server` becomes `server`), joined with
  `|`. Leave it empty in a single-package repo.
- **pre-commit** is the fast check of the staged files: format, lint, type
  check, with autofix where the tools support it. In a Vite+ project
  (`vite-plus` in `package.json`) it runs `vp staged`, with
  `staged: { "*": "vp check --fix" }` in `vite.config.ts`. Elsewhere use
  the project's own staged-files tool if it has one (lint-staged,
  pre-commit and the like). If it has none, skip this hook rather than
  checking the whole repo on every commit.
- **pre-push** is the full check before the code leaves the machine: the
  whole repo's check, tests and build. Chain the scripts the project
  already has into one `ready` script (in a Vite+ monorepo:
  `vp check && vp run -r test && vp run -r build`) and run it from the hook
  (`vp run ready`, `npm run ready`). Leave out steps the project lacks, and
  don't invent new tooling for it.

Write each hook as a shell script starting with `#!/bin/sh` and make it
executable. Only add hooks that are missing; never replace an existing one.

## Install on every clone

- Vite+ project: set the `prepare` script to
  `vp config --no-agent --hooks-dir .hooks` and run it.
- Other project with `package.json`: set `prepare` to
  `git config core.hooksPath .hooks` and run it.
- No `package.json`: run `git config core.hooksPath .hooks` and tell the
  user that every clone needs this command once.

If `prepare` already has a command, append to it with `&&` instead of
replacing it.

## Verify

Run the new hooks once to make sure they pass before committing them. If
the project's checks already fail, show the user the output and leave the
fix to them.
