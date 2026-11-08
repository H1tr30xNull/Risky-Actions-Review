# Risk Rubric

## Score anchors

| Score | Label | Typical actions |
|---|---|---|
| **5** | Critical | Force push / history rewrite on a shared or public branch · `rm -rf` outside scratch dirs · database drop/reset/migration on real data · secrets committed, printed, or sent externally · auth/authorization logic replaced · production deploy · disabling security controls (TLS verify off, auth bypass flags, permission `777`) |
| **4** | High | Dependency installs/upgrades (supply chain, lockfile churn) · CI/CD or deployment config edits · infrastructure-as-code changes · global/user-level config changes (`~/.bashrc`, `~/.claude/settings.json`, hooks, git config) · deleting tracked files · amend/rebase of already-pushed commits · publishing content externally (artifact, gist, package, post) · large generated code in security-sensitive modules |
| **3** | Medium | Non-trivial logic changes in core code without tests run · build script changes · installing user-level tools/skills · new commits on a shared branch · env var reads of sensitive values (not printed) · changes across many files |
| **2** | Low | Local commits on a personal branch · new files in project · docs/README edits · test-only changes · reversible config tweaks in project scope |
| **1** | Trivial | Scratch/temp files · formatting · reading/listing |

## Modifiers

| Condition | Adjust |
|---|---|
| Irreversible or hard to undo | +1 |
| Affects other people / public / production | +1 |
| Security-sensitive area | +1 |
| Not verified (no tests, no diff check) | +1 for logic changes |
| Done in an isolated scratch dir / worktree | −1 |
| Trivially reversible and contained (e.g. untracked new file) | −1 |
| User explicitly asked for it | no change (still report — awareness, not blame) |

Cap at 5, floor at 1.

## Grouping

Merge into one row when actions share purpose and area:
- "Installed 12 npm packages (incl. `axios`, `jsonwebtoken`)"
- "Edited 7 files under `src/auth/`"
- "Created 3 skill folders under `~/.claude/skills/`"

Use the highest score in the group.

## Good vs bad "Why should I care?"

| Bad | Good |
|---|---|
| Modified auth.ts | A mistake in this login check could let strangers in or lock users out. |
| Ran npm install | New packages run code on your machine and can change app behavior. |
| git push --force | The old commit on GitHub was overwritten and is hard to get back. |
| Edited settings.json | This changes how Claude Code behaves in every future session. |
