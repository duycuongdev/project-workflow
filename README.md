# 🛠️ project-workflow

> A battle-tested workflow skill for AI coding assistants (ZCode, Claude Code, Cursor...) that enforces professional engineering discipline on **every** coding task — Git branching, mandatory verification, security checklists, i18n, responsive UI, and CI/CD.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Type: AI Skill](https://img.shields.io/badge/Type-AI%20Skill-8A2BE2)](#what-is-this)
[![Stack](https://img.shields.io/badge/Stack-MERN-61DAFB?logo=react&logoColor=black)](#built-from-a-real-production-project)

**⚔️ Distilled from 5 hardening phases of a real production MERN project — not theory.**

---

## What is this?

This is an [Agent Skill](https://agentskills.io) for AI-powered coding assistants. Once installed, your AI assistant automatically follows a professional engineering workflow on **every coding task** — no more "just committed straight to master", no more "works on my machine", no more forgotten security checks.

**Before this skill** your AI assistant might:
- Commit directly to `master`
- Merge without running tests
- Forget authorization checks on new endpoints
- Hardcode Vietnamese/English strings in the UI
- Skip mobile verification
- Leave test servers running on ports

**After installing**, every task follows the full 8-step discipline below — automatically.

## The 8-step Git workflow

```text
1. Branch from main      →  git checkout -b feat/task-name
2. Code                  →  (apply security + frontend checklists)
3. Verify                →  lint + build + full test suite, ALL green
4. Commit                →  clear message explaining WHAT and WHY
5. Push branch           →  git push -u origin feat/task-name
6. Pull main             →  receive teammates' merges
7. Merge (--no-ff) + push main
8. Delete branch         →  local + remote
```

## What's inside

| Section | Covers |
|---|---|
| **Definition of Done** | 6 mandatory checks before any merge (lint, build, full tests, real-data verification, secrets hygiene, failure fallbacks) |
| **Backend security checklist** | IDOR/authorization service pattern, MongoDB transactions, indexes, rate limiting, anti-enumeration, JWT `jti`, bcrypt async ≥10, hashed token rotation |
| **Frontend checklist** | 100% i18n (zero hardcoded strings — automated scan), responsive at 375px, real-browser verification, contrast variants |
| **Testing standards** | Unit + integration suites committed to the repo, dynamic test data, self-cleanup, self-skip when config is missing |
| **CI/CD rules** | GitHub Actions gotchas (secrets in `if:`, parallel test files, Environment secrets) |
| **10 hard-won pitfalls** | Real bugs that cost real debugging hours — documented so you never repeat them |

## 10 hard-won pitfalls (a taste)

1. **`node --test` runs test FILES in parallel by default** → spawned test servers fight over the same port. Always `--test-concurrency=1`.
2. **Killing test servers on Windows**: `process.kill()` is often not enough — use `taskkill /F /T /PID`.
3. **`secrets` cannot be referenced in GitHub Actions `if:` conditionals** — the run fails with 0 jobs. Let tests self-skip on missing env instead.
4. **JWTs minted in the same second for the same user are IDENTICAL** (same `iat` + payload) → always add `jti: uuidv4()`.
5. **Adding an env var but forgetting the central config mapping** → your "mock email for CI" feature never activates. Grep the config after adding any env var.
6. **`slugify` leaves leading/trailing hyphens** (`'dấu @#'` → `'dau-'`).
7. **`describe.skipIf` doesn't exist in Node 20** — use `const d = cond ? describe : describe.skip`.
8. **Deployments lag silently** — compare the production bundle hash before declaring a feature live.
9. **`Date.now()`-based IDs collide** within the same millisecond.
10. **Environment secrets require the job to declare `environment: <name>`** — otherwise they silently read as empty.

*(The full list with context is in [SKILL.md](SKILL.md#8-hard-won-pitfalls-cost-real-debugging-time-read-to-avoid-repeating).)*

## Built from a real production project

Every rule here was forged while hardening a full-stack **Trello clone**
(MERN + Socket.IO real-time + i18n EN/VI + Cloudinary + Brevo) through 5 phases:

| Phase | What happened |
|---|---|
| 1. Security | Fixed IDOR on all endpoints, socket authentication, XSS, token rotation + theft detection |
| 2. Data integrity | MongoDB transactions on 11 multi-document ops, 13+ indexes, UUID migration |
| 3. Product | Forgot/reset password, i18n EN/VI (363 keys × 2), Swagger, health check |
| 4. Quality | 43 automated tests, CI pipeline, UI polish, responsive audit |
| 5. CI | GitHub Actions green — including fixing 4 layers of CI issues live |

**Result: 43/43 tests passing, CI green, deployed to production.** The workflow is what made that possible.

## Install

### ZCode

Copy the skill into your user skills directory:

```bash
git clone https://github.com/duycuongdev/project-workflow.git \
  ~/.agents/skills/project-workflow
```

The skill auto-triggers on any coding task. You can also force it:

```text
/project-workflow add a comment feature to the board
```

### Other AI assistants (Cursor, Claude Code, ...)

Open `SKILL.md` and paste its rules into your assistant's system prompt / rules file
(`.cursorrules`, `CLAUDE.md`, ...). The workflow is plain markdown — no lock-in.

## Repository layout

```text
project-workflow/
├── SKILL.md      # The skill itself (AI-readable workflow + checklists)
├── README.md     # This file
└── LICENSE       # MIT
```

## Why a skill instead of a wiki page?

A wiki page gets read once. A skill gets **injected into the AI's context on every
relevant task** — the discipline is enforced by the assistant itself, at the moment
code is written, not documented for humans to forget.

## Contributing

PRs welcome! If you hit a pitfall that cost you debugging time, add it to section 8
with the fix — that's the heart of this skill.

## License

[MIT](LICENSE) © 2026 [duycuongdev](https://github.com/duycuongdev)
