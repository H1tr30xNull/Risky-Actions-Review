---
name: risky-actions
description: Risky Actions Review — review the current Claude Code session and surface the top-K (default 4) highest-risk actions the agent performed, each with a one-sentence "Why should I care?". Use when the user invokes /risky-actions, or says "review session risks", "what are the riskiest things you did?", "what did you just do?", "what changed?", "anything dangerous?", "danger audit", "risk radar", "what should I double-check?", or asks for a risk recap before committing, pushing, deploying, or ending a session. Read-only: it reports, it never undoes or fixes anything unless asked.
---

# Risky Actions Review

## Purpose

Review what the agent did in this session and point the user at the few actions most
worth a second look — changes that could affect source code, configuration, credentials,
security, data integrity, or production behavior.

This is about **user awareness, not blame**. The user is busy; they won't re-read a long
transcript. A short, ranked, plain-language list lets them spend two minutes reviewing the
right things instead of everything.

The review is **read-only**. Don't revert, fix, or "clean up" anything while reviewing —
that would itself be an unreviewed action. Offer fixes at the end; act only if the user asks.

---

## Phase 1 — Session Inventory

Build an action log of everything the agent did that touched the outside world.

**Where to look** (use all that apply):
1. The conversation itself — every tool call and its result in this session.
2. If the context was summarized/compacted, the raw transcript:
   `~/.claude/projects/<project-dir>/*.jsonl` (most recent file for this session).
3. Ground truth in the workspace: `git status`, `git diff --stat`, `git log --oneline -n 20`,
   `git reflog -n 20` (reveals amends, resets, force pushes) for every repo touched.

**What counts as an action:** shell commands · file creations, modifications, deletions ·
dependency changes and package installs · git operations (commit, amend, rebase, reset,
push, force push, branch delete) · configuration updates (incl. `~/.claude/` settings,
hooks, skills, memory) · environment variable / secret access · code generation in
sensitive areas · build, migration, deployment commands · anything published or sent
externally (web publishes, API calls, uploads, messages, PR comments).

Pure reads (viewing files, listing directories, searches) are not actions unless they
exposed secrets (e.g. printing a `.env` or token into the transcript).

For each action capture:

| Field | Description |
|---|---|
| Action | What happened |
| Scope | What was affected |
| Artifacts | Files, directories, repos, services |
| Order | Relative position in the session (#1, #2, …) |
| Potential impact | Initial assessment |

Keep this log as working notes; show it only if the user asks for the full log.

---

## Phase 2 — Risk Scoring

Score each action **1–5** using the factors below. Details and examples are in
`references/risk-rubric.md`.

| Score | Label |
|---|---|
| 5 | Critical |
| 4 | High |
| 3 | Medium |
| 2 | Low |
| 1 | Trivial |

Risk factors:
- **Destructive operations** — `rm -rf`, file deletion, database reset, overwrite
- **Security impact** — authentication/authorization, permissions, credentials, secrets, encryption
- **System impact** — dependency upgrades, environment changes, build/CI changes, global config
- **Irreversibility** — history rewrite, force push, data overwrite, external publish/send
- **Blast radius** — single file → repository → whole environment / production / other people

Raise the score when factors combine (an irreversible change with a wide blast radius is
worse than either alone). Lower it when the action is easily undone and contained
(e.g. a new file in a scratch directory). Was it verified (tests run, diff inspected)?
Unverified changes in sensitive areas deserve a higher score.

---

## Phase 3 — Top-K Selection

Default **K = 4**. The user can set another value ("top 10", "/risky-actions 6").

- Pick the highest-scoring actions.
- **Group** repeated or related actions into one row (e.g. "Edited 7 files in `auth/`",
  "Installed 12 npm packages").
- If fewer than K actions are review-worthy, show fewer — don't pad with trivia.
- If nothing scored above Low, say so plainly: the session was low-risk.

---

## Output

Use this format (template with more examples in `references/output-template.md`):

```markdown
## Top Risky Actions

| Rank | Risk | Action | Why should I care? |
|---|---|---|---|
| 1 | Critical | Force-pushed `main` on Paper-Analyzer- | The previous commit was replaced on GitHub and is now hard to recover. |
| 2 | High | Replaced JWT validation logic in `auth/verify.ts` | A mistake here could let unauthenticated users in or lock everyone out. |
| … | | | |

**Session summary:** <one line — where most of the risk came from>.

**Recommended review:** (only if relevant)
- `auth/verify.ts`
- `package-lock.json`
```

Match the user's language for the prose (e.g. Vietnamese if they write Vietnamese);
keep commands, paths, and code identifiers as-is.

---

## Core Rules

1. **Impact over quantity.** Ten small edits are often less important than one
   authentication change.
2. **Prefer security-sensitive changes.** Authentication, authorization, secrets,
   encryption, deployment, infrastructure rank above everything else at equal score.
3. **Explain risk in human language.** Not "Modified auth.ts" but "Modified
   authentication logic, which could affect who can access the app."
4. **Don't flag every action.** Only the most review-worthy ones.
5. **Always answer "Why should I care?"** in one concise sentence per row.
6. **Be honest about yourself.** Include actions the user asked for — risk doesn't
   disappear because it was requested — and don't downplay your own mistakes.
   Only report what actually happened; if something is uncertain (e.g. you can't tell
   whether a push succeeded), say so instead of guessing.
