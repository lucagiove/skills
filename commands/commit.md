---
description: Draft Conventional Commit message(s); supports workspaces with multiple git repositories
---

You are drafting a git commit message for this change set following **Commit standards** (Conventional Commits: type, optional scope, imperative description).

## Standards

- **Write the intent, not the diff**: the message must say **why this change exists** — the problem it fixes, the goal it serves, or the behavior it unlocks. Do **not** restate which files, fields, or lines moved; the reader has the diff for that.
- **Use the words the user and the code use**: pick terms from the user's prompt, the code, the domain, or recent commits in the repo. Avoid uncommon, abstract, or fancy words when a plain one works.
- **Prefer a single-line commit**: if the change is small and the subject already explains the intent, stop there. Do **not** add a body just to look thorough.
- **Add a body only when needed**: use it when the subject cannot fit the reason, when there is a non-obvious trade-off, or when reviewers need migration notes.
- **Types** (pick one): `build`, `ci`, `docs`, `feat`, `fix`, `perf`, `refactor`, `style`, `test`, `chore`.
- **Subject**: imperative mood (e.g. "add ...", "fix ...", "restore ..."). It must answer "what intent does applying this commit serve?", not "what lines moved?".
- **With scope**: `<type>(<scope>): <description>` — **without**: `<type>: <description>`.
- **Scope**: optional but use it when it clarifies the area touched. Lowercase, short. **Must be derived from an actual folder / module or a term that already appears in recent `git log` entries for that repo.** Never invent a scope that doesn't map to a real module or established convention in the codebase. When uncertain, run `git log --oneline -20` to mirror existing scope style, or omit the scope entirely.
- **Breaking changes**: if a shared library or API is not backward compatible, add a footer after a blank line:

  ```text
  BREAKING CHANGE: <what changed and how callers migrate>
  ```

## Subject shape for `fix`

A `fix` subject reads as: `fix(<scope>): <what is broken> <reason it broke> [<how it is fixed>]`

The `how` is optional — skip it when the diff makes it obvious. Add a body only if the reason or fix is too long for one line.

**Do:**

```text
fix(prints): broken receipt preview due to not backward compatible data
```

**Don't:**

```text
fix(prints): expose legacy appliedDiscount.discount in receipt preview data
```

The "don't" example describes the patch (which field was exposed). The "do" example describes the intent (the preview was broken, and why).

## Multi-repository workspaces

The workspace may contain more than one git repository, and the workspace root itself may not be a git repo.

- **Discover roots**: if `git status` fails with "not a git repository", find each repo (directories with a `.git`, or paths the user names) and treat each as a separate target.
- **Inspect per repo**: run the inspect command **once from each repo root** that changed.
- **Check current branch per repo**: read it from the `On branch <name>` line of each repo's `git status`. If the dirty repos are **not all on the same branch**, **warn the user** with the per-repo `repo → branch` mapping and pause for confirmation before committing.
- **One commit per repo**: each repo gets its own message; never bundle unrelated repos.
- **Coordinated changes**: when one feature spans multiple repos, align wording across messages and call out merge or deploy order if it matters (e.g. a breaking API before its clients).
- **Commit all repos**: when the user asks to commit everything, iterate each dirty repo. `git add` and `git commit` are both allowed inside the explicit "commit" request, run from each repo's root.

## Allowed commands

To keep this prompt fast and easy to whitelist, prefer these patterns. Do **not** invent variants (`git diff <path>`, `git show`, `git log -p`, etc.).

1. **Inspect** (per git repository root):

   ```bash
   git status && git diff HEAD
   ```

   `cd` into each repo root first if the workspace root is not a git repository.

2. **Mirror repo style** (optional, only when the type/scope/wording is unclear from the chat):

   ```bash
   git log --oneline -20
   ```

3. **Commit** (only after the user has approved the message(s)). Use a HEREDOC so multi-line bodies survive shell quoting:

   ```bash
   git commit -m "$(cat <<'EOF'
   <subject>

   <optional body>

   <optional BREAKING CHANGE: ...>
   EOF
   )"
   ```

For multiple repositories, repeat staging and the commit step inside each repo's root.

Anything else (amend, push, rebase, etc.) requires an **explicit** request from the user.

## Workflow

1. **Infer intent primarily from this chat** — what the user asked for, the constraints they gave, the outcome they wanted. This is the main source for the message. When the work evolved through refactors or review, trace back to the original goal that motivated it; the most recent turns usually describe mechanism (how), not purpose (why).

2. **Identify git root(s)** — single repo or several. Run the inspect command from each repo that changed to confirm the diff and spot mixed concerns. Skip only when the chat already makes every change set fully clear.

3. Choose **type** from the nature and intent of the change (not from filename patterns).

4. Choose **scope** when it narrows the intent to a module or feature; skip it when it adds noise.

5. Draft the **subject** (per repo if multi-repo). Keep it on one line. Reuse the user's and the code's own words.

6. Add a **body** or `BREAKING CHANGE` footer **only if** the subject cannot carry the intent on its own.

7. **Only ask the user if the reason of the change is unclear** and the chat plus the diff do not give enough context to write an honest subject. Otherwise, draft and propose.

8. For multi-repo changes, show all proposed messages **labeled by repo path** before any commit, unless the user already approved a batch commit.

## Output

- **Always show the full proposed commit message(s)** to the user (subject, plus body and `BREAKING CHANGE` if any) in clear, copy-ready blocks **before** running the commit command — **one block per repository** when several git roots changed.
- If **one repository** bundles unrelated intents, recommend **multiple commits** in that repo and give a suggested message for each.
- **STOP after proposing. Never run `git add` or `git commit` until the user replies with an explicit confirmation** (e.g. "go", "yes", "ok", "do it"). Do not treat silence, a follow-up question, or an unrelated message as confirmation.
- **Never run `git push`** (or any remote-upload equivalent) unless the user **explicitly** asks you to push.
