---
name: "dev-habits"
description: "Encodes my git habits. Invoke when user says '提交git', '打tag/tag', or '重新tag/重打/retag' for local commits, tagging, version bumping, and optional tag pushes."
---

# Dev Habits (Git)

This skill captures my personal git workflow conventions.

## When to Invoke

- The user says "提交git" or "提交到git".
- The user says "打tag <tag> ..." or "tag <tag> ...".
- The user says "重新tag <tag>".
- The user says "重打 <tag>" or "retag <tag>".

## Convention: "提交git"

Interpretation:

- "提交git" means **commit locally only**.
- **Do not push** to any remote unless explicitly asked.
- Commit message must be **English**, **≤ 20 words**.

Workflow:

1. Confirm the intended scope (files/stage status) if ambiguous.
2. Stage changes intentionally (avoid `git add -A` unless requested).
3. Create a local commit with an English message ≤ 20 words.
4. Show the resulting commit hash and summary.
5. Do not run any push commands.

## Convention: "重新tag"

Interpretation:

- "重新tag" means: retag an existing tag name to point to a desired commit, and **push the updated tag to a specified remote**.
- If the remote already has the tag, delete the remote tag first, then push the new tag.

Required inputs (ask if missing):

- Tag name (e.g., `v1.2.3`)
- Target commit (SHA / branch / HEAD)
- Remote name (e.g., `origin`, `upstream`)
- Tag type preference: annotated (default) or lightweight

Safe default behavior:

- Prefer **annotated tags**.
- Never rewrite branches; only operate on tags.

Command outline:

1. Verify current tag target:
   - `git show-ref --tags <tag>`
   - `git rev-parse <target>`
2. Delete local tag (if exists):
   - `git tag -d <tag>`
3. Delete remote tag (if exists):
   - `git push <remote> --delete <tag>`
4. Recreate tag:
   - Annotated: `git tag -a <tag> <target> -m "<message>"`
   - Lightweight: `git tag <tag> <target>`
5. Push tag to the specified remote:
   - `git push <remote> <tag>`

Notes:

- Keep tag messages short and in English unless the user requests otherwise.
- If the user provides a remote URL instead of a remote name, add it as a temporary remote only when explicitly requested.

## Convention: "打tag" / "tag"

Interpretation:

- "打tag <tag> <message...>" or "tag <tag> <message...>" means: create a git tag.
- The content after the tag name is used as the tag message (English preferred).
- Default to **annotated tags** unless the user explicitly requests lightweight tags.
- If this is a JavaScript app and `package.json` exists, extract the version from the tag name and update `package.json` accordingly.
- If a `VERSION` file exists in the project root, update it to the tag value by default (unless the user explicitly says not to), **commit it**, then create the tag.

Tag parsing:

- Tag name is the first token after "打tag" / "tag".
- Tag message is the remaining text after the tag name (can be empty).
- Version extraction for JS: remove a leading `v` if present (e.g., `v1.2.3` -> `1.2.3`), then use the rest as the version string.

Workflow:

1. Verify working tree status and target commit (default: `HEAD`).
2. Prepare version files (unless the user opts out of specific steps):
   - If a `VERSION` file exists:
     - Replace its contents with the tag value (exact tag string).
     - Keep a trailing newline.
   - If `package.json` exists:
   - Update `version` using the extracted version from the tag.
   - Update the lockfile if the project uses one (e.g., `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`) and it is affected.
3. Commit version changes locally (required when any version file was modified):
   - Follow the "提交git" convention (local commit only, no push, English ≤ 20 words).
   - Default commit message: `Bump version to <tag>`.
4. Create the tag:
   - Annotated: `git tag -a <tag> <target> -m "<message>"`
   - Lightweight: `git tag <tag> <target>`
5. Do not push tags unless explicitly asked or a remote is specified.

## Convention: "重打" / "retag"

Interpretation:

- "重打 <tag>" or "retag <tag>" means: overwrite an existing tag name to point to the desired commit (default: `HEAD`).
- Behavior matches "重新tag", except pushing is only performed when the user explicitly asks or provides the remote.

Workflow:

1. Delete local tag if it exists: `git tag -d <tag>`
2. Recreate the tag pointing to the desired commit (annotated by default).
3. If a remote is specified or the user asks to push:
   - Delete the remote tag if it exists: `git push <remote> --delete <tag>`
   - Push the updated tag: `git push <remote> <tag>`
