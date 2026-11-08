# Output Template

```markdown
## Top Risky Actions

| Rank | Risk | Action | Why should I care? |
|---|---|---|---|
| 1 | Critical | Force-pushed branch history | Previous commits may be difficult to recover. |
| 2 | High | Replaced JWT validation logic | Authentication failures or bypasses could be introduced. |
| 3 | High | Installed new npm dependencies | New packages introduce supply-chain and compatibility risks. |
| 4 | Medium | Modified CI/CD workflow | Future deployments may behave differently than expected. |

**Session summary:** Most of the risk in this session came from authentication, dependency, and deployment-related changes.

**Recommended review:**
- `src/auth/jwt.ts`
- `package-lock.json`
- `.github/workflows/deploy.yml`
```

## Low-risk session

```markdown
## Top Risky Actions

Nothing in this session rose above **Low** risk.

| Rank | Risk | Action | Why should I care? |
|---|---|---|---|
| 1 | Low | Created 2 new docs files | Easy to review or delete; nothing else depends on them. |

**Session summary:** Only new, self-contained files were added — no deletions, config, dependency, or git history changes.
```

## Optional: full log (only when asked)

| # | Action | Scope | Artifacts | Score | Impact |
|---|---|---|---|---|---|
| 1 | … | … | … | 2 | … |
