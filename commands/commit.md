---
description: Draft a Conventional Commit message
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
- **Scope**: optional but use it when it clarifies the area touched. Lowercase, short, taken from the module or feature name in the codebase.
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

## Allowed commands

To keep this prompt fast and easy to whitelist, only these exact commands may be run. Do **not** invent variants (no `git add`, no `git diff <path>`, no `git show`, no `git log -p`, etc.).

1. **Inspect** (one command, always safe):

   ```bash
   git status && git diff HEAD
   ```

   This covers tracked changes (staged + unstaged) and lists untracked files. Run it **at most once** per draft.

2. **Mirror repo style** (optional, only when the type/scope/wording is unclear from the chat):

   ```bash
   git log --oneline -20
   ```

3. **Commit** (only after the user has approved the message). Use a HEREDOC so multi-line bodies survive shell quoting:

   ```bash
   git commit -m "$(cat <<'EOF'
   <subject>

   <optional body>

   <optional BREAKING CHANGE: ...>
   EOF
   )"
   ```

Anything beyond this list (staging files, amending, pushing, rebasing, etc.) requires an **explicit** request from the user.

## Workflow

1. **Infer intent primarily from this chat** — what the user asked for, the constraints they gave, the outcome they wanted. This is the main source for the message.

2. Run the single inspect command above to confirm what actually changed and to spot mixed concerns. Skip it only if the chat already makes the change set fully clear.

3. Choose **type** from the nature and intent of the change (not from filename patterns).

4. Choose **scope** when it narrows the intent to a module or feature; skip it when it adds noise.

5. Draft the **subject**. Keep it on one line. Reuse the user's and the code's own words.

6. Add a **body** or `BREAKING CHANGE` footer **only if** the subject cannot carry the intent on its own.

7. **Only ask the user if the reason of the change is unclear** and the chat plus the diff do not give enough context to write an honest subject. Otherwise, draft and propose.

## Output

- **Always show the full proposed commit message to the user** (subject, plus body and `BREAKING CHANGE` if any) in a clear, copy-ready block **before** running the commit command. Pause for confirmation if the workflow is ambiguous; never commit silently.
- If the changes bundle unrelated intents, recommend **multiple commits** and give a suggested message for each.
- Run the commit command **only after** the user has seen the message and explicitly asked you to perform the commit (or has clearly confirmed to proceed).
- **Never run `git push`** (or any remote-upload equivalent) unless the user **explicitly** asks you to push.
