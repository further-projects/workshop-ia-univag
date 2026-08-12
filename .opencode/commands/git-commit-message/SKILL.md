---
name: git-commit-message
description: Analyze repository changes, divide them into small related groups, and prepare or execute English Conventional Commits. Use when Codex needs to inspect changes before committing, propose commit boundaries, stage related files selectively, write or validate commit messages, or run git commit after explicit user approval.
---

# Git Commit Message

Create focused Git commits using Conventional Commits. Keep messages in English and never combine unrelated changes in one commit.

## Workflow

1. Inspect `git status`, the unstaged diff, and the staged diff before proposing a commit.
2. Identify logical change groups by purpose, feature, fix, or maintenance concern.
3. Flag sensitive, generated, temporary, or suspicious files before staging them.
4. Propose an ordered commit plan containing:
    - files or diff sections belonging to each commit;
    - the reason those changes belong together;
    - the proposed Conventional Commit message.
5. Ask for explicit user approval before staging files or diff sections for a commit.
6. Stage only the approved group. Use selective or patch staging when a file contains unrelated changes.
7. Review the staged diff and confirm it matches the approved group.
8. Ask for explicit user approval immediately before executing each `git commit`.
9. Commit only the reviewed staged changes, then inspect repository status before continuing.

Never stage the entire repository merely for convenience. Never use `git add .`, `git add -A`, or an equivalent broad command unless the user explicitly requests it after reviewing the affected paths.

## Message Format

Use:

```text
<type>[scope]: <description>

[body]
```

Keep the subject concise, imperative, lowercase after the colon, and without a trailing period. Describe the outcome of the change rather than the activity performed.

Use a scope only when it adds useful project context, such as `web`, `api`, `auth`, `editor`, `schemas`, `config`, or `deps`. Do not invent a scope solely to fill the field.

## Commit Types

- `feat`: add or change user-facing functionality.
- `fix`: correct faulty behavior.
- `docs`: change documentation only.
- `refactor`: restructure code without changing expected behavior.
- `test`: add or change tests without changing production behavior.
- `build`: change the build system, packaging, or dependencies.
- `ci`: change continuous integration configuration.
- `chore`: perform repository maintenance not covered elsewhere.
- `perf`: improve performance without changing expected behavior.
- `style`: change formatting without affecting behavior.
- `revert`: revert a previous commit.

Choose the type from the primary purpose of the staged change, not from the filenames involved.

## Commit Boundaries

Place changes in the same commit only when they form one coherent, reviewable unit. A commit should be safe to review and, when practical, revert independently.

Separate commits when changes:

- address different features or bugs;
- mix refactoring with behavior changes that can stand alone;
- combine dependency or tooling updates with unrelated application work;
- change documentation unrelated to the implementation;
- contain independent tests or cleanup for another concern.

Keep tests with the implementation they verify when both represent the same functional change. Keep required schema, migration, configuration, or contract updates with their corresponding implementation when separating them would leave the repository inconsistent.

Do not split a change merely to minimize file count. Prefer the smallest coherent commit, not the smallest possible diff.

## Commit Body

Omit the body when the subject fully explains a simple change. Add a body when context, motivation, constraints, or non-obvious behavior would help reviewers.

Write the body in English, use complete sentences, and explain why the change was needed or what important consequence it has. Do not repeat the subject.

## Breaking Changes

Mark an intentional breaking change with `!` after the type or scope and add a `BREAKING CHANGE:` footer describing the migration impact.

```text
feat(api)!: replace document content contract

BREAKING CHANGE: clients must send Tiptap JSON instead of HTML.
```

Do not infer a breaking change from uncertainty. Explain the evidence and ask the user when compatibility impact is unclear.

## Safety Rules

- Never commit secrets, credentials, environment files, private keys, or unexpected binary artifacts.
- Preserve pre-existing staged changes; do not unstage or rewrite them without explicit approval.
- Do not amend, rebase, reset, force-push, or bypass hooks unless the user explicitly requests the specific action.
- Do not use `--no-verify` unless the user explicitly authorizes bypassing hooks after seeing the failure.
- If hooks modify files, stop and review the resulting diff before proceeding.
- Report validation or hook failures without committing additional changes automatically.

## Examples

```text
feat(editor): add document heading navigation
fix(auth): enforce login attempt lockout
docs: clarify local development setup
refactor(api): extract document ownership guard
test(auth): cover expired session rejection
build(deps): update workspace dependency catalog
```

For a change needing context:

```text
fix(auth): keep active sessions valid after page reload

Read the session from the server-managed cookie so protected routes do not
depend on client state initialization.
```
