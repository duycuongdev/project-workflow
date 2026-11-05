---
name: project-workflow
description: Complete battle-tested workflow for ANY coding task or feature in a repository (especially MERN: Node/Express/MongoDB/React/Vite/Socket.IO). Use ALWAYS when the user asks to modify code, add a feature, fix a bug, refactor, write tests, or touch the repo in any way — even if they don't mention "workflow" or "process". Covers per-task Git branching, mandatory pre-merge testing, security checklists, i18n, responsive verification, and CI/CD.
---

# Project Workflow (battle-tested)

This workflow was distilled from a real production MERN project (a Trello clone) across
5 hardening phases: security, data integrity, product features, and quality. **Follow it
completely for every task, no matter how small.**

## 1. Core principles

- **One task = one branch**, named by type: `feat/`, `fix/`, `refactor/`, `chore/`, `perf/`, `ci/` + short description (e.g. `fix/idor-board-authorization`).
- **Never commit directly to the main branch** (`master`/`main`).
- **Never merge without evidence**: tests must run and pass, lint + build must be green.
- After merging, **delete the branch both locally and remotely** — no stale branches.

## 2. The 8-step Git workflow (perform IN ORDER for every task)

```bash
# 1. Branch from the main branch (pull first to ensure it is fresh)
git checkout master && git pull origin master
git checkout -b <type>/task-name

# 2. Code (apply the checklists in sections 3-6 while coding)

# 3. Verify: lint + build + tests ALL green before committing
npm run lint && npm run build && npm test

# 4. Commit (clear message: type + description + body explaining why)
git add -A && git commit -m "feat(scope): short description

- change detail 1
- change detail 2"

# 5. Push the branch to remote
git push -u origin <type>/task-name

# 6. Switch back to main + pull (receive other people's merges)
git checkout master && git pull origin master

# 7. Merge (--no-ff to preserve history) + push main
git merge <type>/task-name --no-ff -m "merge: <type>/task-name into master"
git push origin master

# 8. Delete the branch (local + remote)
git branch -d <type>/task-name
git push origin --delete <type>/task-name
```

If `git push origin master` is rejected — the remote has newer commits: `git pull`,
merge again, and push. NEVER use `--force` on shared branches.

## 3. Definition of Done — mandatory before merging

- [ ] Lint exits 0 (fix warnings too, never loosen `--max-warnings`)
- [ ] Build succeeds
- [ ] Tests pass (re-run the WHOLE suite, not just the new tests)
- [ ] Verified against real data/behavior (don't trust code review alone — smoke test through the API/browser)
- [ ] No secrets/keys in code or commits (`.env` gitignored, `.env.example` maintained)
- [ ] Failure fallbacks thought through (what happens if this step fails midway?)

## 4. Security checklist (Backend — every feature touching data)

- [ ] **Authorization**: every read/write endpoint verifies the user belongs to the resource (blocks IDOR).
      Extract into a dedicated service (e.g. `authorizationService`) instead of scattered if-else checks.
- [ ] **Multi-document writes in a transaction** (MongoDB `withTransaction`) — prevents orphaned data.
- [ ] **Index** every frequently queried field; unique index for fields that must not duplicate.
- [ ] **Rate limiter** for sensitive public endpoints (login/register/email/forgot-password).
- [ ] **Anti-enumeration**: public endpoints return the SAME response whether the resource exists or not.
- [ ] Secrets via env only; JWTs include `jti` so tokens minted in the same millisecond never collide.
- [ ] bcrypt async with cost ≥ 10 (never hashSync — it blocks the event loop).
- [ ] Input validation at the route layer (Joi) + escape/sanitize everything rendered back.
- [ ] Expired tokens must be revocable + rotated (store HASHED tokens, never raw tokens).

## 5. Frontend checklist (every UI change)

- [ ] **i18n 100%**: every UI string goes through `t('key')` — NEVER hardcode Vietnamese or English.
      Before merging, run an automated scan for non-ASCII (Vietnamese) characters in src
      (excluding comments) — count must be 0.
- [ ] **Responsive at 375px**: verify every new/modified screen in a real browser (or viewport tool).
- [ ] **Verify in a real browser** (don't trust code review): screenshot/inspect screens, click the main flows.
- [ ] Contrast: text must be readable on EVERY background (watch for white-on-white when you
      hardcode white text for an appbar then reuse the component on a white page — use a
      `variant` prop + theme-aware colors).
- [ ] No leftover side effects: socket listeners/timers must be removed in the useEffect cleanup.

## 6. Testing (commit tests to the repo, no throwaway scripts)

- **Unit tests** for utils/validations (fast, run on every push).
- **Integration tests** for critical flows (auth, security, multi-step flows) — test data is
  created + cleaned up in before/after hooks, NEVER dependent on hardcoded IDs.
- Scripts: `test` (whole suite, `--test-concurrency=1` so files don't fight over ports),
  `test:e2e` (integration only).
- Tests must self-skip when environment config is missing (e.g. `hasDbConfig`) — never fail
  spuriously on machines without a DB.

## 7. CI/CD

- GitHub Actions: lint + build + test on every push/PR to the main branch.
- **CD auto-deploys** (Render/Vercel) when the main branch is pushed — after deploying,
  verify the health endpoint (`/v1/health` pings the real DB, 503 when the DB is down).
- If tests need a secret: declare env in the workflow, tests **self-skip when absent** —
  never reference secrets inside `if:` (GitHub rejects the workflow → run fails with 0 jobs).
- Secrets stored under **Environment secrets** require the job to declare `environment: <name>`.

## 8. Hard-won pitfalls (cost real debugging time — read to avoid repeating)

1. **`node --test` runs test FILES in parallel by default** → multiple spawned test servers
   fight over the same port. Always add `--test-concurrency=1` when suites spawn servers.
2. **Killing test servers on Windows**: `process.kill()` is often not enough — use
   `taskkill /F /T /PID`, and clean stale ports before re-running.
3. **`describe.skipIf` doesn't exist in Node 20** — use `const d = cond ? describe : describe.skip`.
4. **`secrets` cannot be referenced directly in GitHub Actions `if:` conditionals** — the run
   fails with 0 jobs. Let tests self-skip based on env instead.
5. **JWTs minted in the same second for the same user are IDENTICAL** (same iat + payload) →
   always add `jti: uuidv4()` to the payload; enforce a unique index where tokens are stored.
6. **`Date.now()`-based IDs collide within the same millisecond** → use UUID v4
   (or `crypto.randomUUID()`).
7. **Adding a new env var but forgetting to map it in the central config** (e.g. adding
   `EMAIL_MODE` to env but forgetting `environment.js`) → the "mock for CI" feature never
   activates. After adding an env var: grep the central config + update `.env.example`.
8. **`slugify` leaves leading/trailing hyphens** (`'dấu @#'` → `'dau-'`) — trim hyphens at both ends.
9. **A lint gate with `--max-warnings 0`** means even warnings block the merge — clean up
   at the root instead of loosening the gate.
10. **Deploys can lag silently**: after pushing, verify the production bundle actually changed
    (compare the hashed filename in index.html) before declaring the feature live.

## 9. When working on UI — review with your eyes before reporting done

- Use a real browser (in-app browser / viewport tool) to open each new/modified screen:
  desktop + mobile 375px.
- Catch what code review can't: white text on white background, floating label duplicating
  the placeholder, orphaned pagination, duplicated avatars, contrast over dynamic photo
  backgrounds.
- Make CI failures readable via the API: don't use `secrets` in `if:`, surface failures as
  annotations (`::error::`) or job summary for remote debugging.

## 10. End of every session

- Summarize for the user: what was done, the evidence (test counts, health URL, screenshots),
  and any remaining user actions (e.g. configuring a dashboard secret).
- If a pattern repeats (the user asks for the same thing ≥ 2 times) — propose turning it into a skill.
