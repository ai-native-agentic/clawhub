# AGENTS.md — clawhub

**Branch:** main @ 33a7cec  
**Generated:** 2026-03-17

## OVERVIEW

clawhub is the skill registry and content hub for the openclaw ecosystem. It provides a centralized platform for discovering, publishing, and managing AI agent skills at https://clawhub.ai. The project is built on TanStack Start (frontend) and Convex (serverless backend), with 437 files spanning 71K LOC.

**Core capabilities:**
- Skill publishing and versioning (semantic versioning, changelog tracking)
- Skill discovery (TF-IDF search, trending leaderboards, category filtering)
- User authentication (GitHub OAuth via @convex-dev/auth)
- Skill ownership and transfers (ownership verification, transfer requests)
- Download tracking and install analytics (per-skill, per-version, per-user)
- Star/comment system (engagement metrics, moderation)
- GitHub backup integration (automatic skill backups to GitHub repos)
- Moderation tools (manual overrides, suspicious content detection, staff audit logs)
- API endpoints (HTTP API v1 for CLI integration, skill file serving)

**Tech stack:**
- Frontend: TanStack Start (React 19, TanStack Router, Tailwind CSS 4, Monaco Editor)
- Backend: Convex serverless functions (queries, mutations, actions, HTTP routes)
- Linting: Biome + oxlint (type-aware)
- Testing: Vitest 4 + jsdom (80% coverage target)
- Package manager: bun (workspace monorepo)

**URL format:** https://clawhub.ai/{owner}/{slug}

## STRUCTURE

```
clawhub/
├── src/                          # TanStack Start frontend
│   ├── routes/                   # File-based routing
│   │   ├── index.tsx             # Homepage
│   │   ├── skills/index.tsx      # Skills browse page
│   │   ├── $owner/$slug.tsx      # Skill detail page
│   │   ├── upload.tsx            # Skill upload page
│   │   ├── settings.tsx          # User settings
│   │   ├── dashboard.tsx         # User dashboard
│   │   ├── admin.tsx             # Admin panel
│   │   ├── search.tsx            # Search results
│   │   ├── stars.tsx             # Starred skills
│   │   ├── u/$handle.tsx         # User profile
│   │   └── cli/auth.tsx          # CLI authentication
│   ├── components/               # React components
│   ├── lib/                      # Shared utilities
│   └── styles/                   # Tailwind CSS
├── convex/                       # Convex backend
│   ├── schema.ts                 # Database schema (skills, users, versions, comments, stars, etc.)
│   ├── skills.ts                 # Skill queries/mutations
│   ├── users.ts                  # User queries/mutations
│   ├── search.ts                 # TF-IDF search implementation
│   ├── leaderboards.ts           # Trending/popular leaderboards
│   ├── downloads.ts              # Download tracking
│   ├── stars.ts                  # Star system
│   ├── comments.ts               # Comment system
│   ├── skillTransfers.ts         # Ownership transfer logic
│   ├── githubBackups.ts          # GitHub backup integration
│   ├── httpApi.ts                # HTTP API v1 routes
│   ├── httpApiV1.ts              # HTTP API v1 handlers
│   ├── crons.ts                  # Scheduled jobs (leaderboard rebuild, stats maintenance)
│   ├── maintenance.ts            # Backfill/migration functions
│   ├── auth.ts                   # Authentication logic
│   ├── auth.config.ts            # Auth configuration
│   └── lib/                      # Shared backend utilities
├── convex/_generated/            # Auto-generated Convex API/types (committed for builds)
├── packages/                     # Workspace packages
│   ├── schema/                   # Shared schema types
│   └── clawdhub/                 # Shared utilities
├── docs/                         # Product/spec documentation
├── public/                       # Static assets
├── scripts/                      # Build/maintenance scripts
├── package.json                  # Workspace root
├── vitest.config.ts              # Vitest configuration
├── vitest.e2e.config.ts          # E2E test configuration
└── tsconfig.json                 # TypeScript configuration

Config: .env.local (never commit)
Convex env: JWT keys, GitHub OAuth credentials
Vercel env: VITE_CONVEX_URL, VITE_CONVEX_SITE_URL
```

## WHERE TO LOOK

| Task | Path | Notes |
|------|------|-------|
| Database schema | convex/schema.ts | Skills, users, skillVersions, comments, stars, downloads, skillTransfers, etc. |
| Skill publishing | convex/skills.ts | publishVersion, updateSkill, deleteSkill, transferOwnership |
| Skill search | convex/search.ts | TF-IDF search, skill indexing, search ranking |
| Leaderboards | convex/leaderboards.ts | Trending/popular leaderboards, rebuild logic |
| Download tracking | convex/downloads.ts | recordDownload, getDownloadStats |
| Star system | convex/stars.ts | toggleStar, getStarCount, getUserStars |
| Comment system | convex/comments.ts | addComment, editComment, deleteComment, moderation |
| Ownership transfers | convex/skillTransfers.ts | requestTransfer, acceptTransfer, cancelTransfer |
| GitHub backups | convex/githubBackups.ts | Automatic skill backups to GitHub repos |
| HTTP API v1 | convex/httpApi.ts, convex/httpApiV1.ts | /api/v1/skills/{slug}, /api/v1/skills/{slug}/file |
| Authentication | convex/auth.ts, convex/auth.config.ts | GitHub OAuth, session management |
| Cron jobs | convex/crons.ts | Leaderboard rebuild, stats maintenance |
| Maintenance | convex/maintenance.ts | Backfill functions, data migrations |
| Frontend routes | src/routes/ | File-based routing (TanStack Router) |
| Components | src/components/ | Reusable React components |
| Utilities | src/lib/, convex/lib/ | Shared utilities (frontend/backend) |
| Tests | convex/**/*.test.ts, src/**/*.test.ts | Vitest tests (80% coverage target) |
| E2E tests | e2e/ | Playwright E2E tests |
| Build scripts | scripts/ | copy-og-assets.ts, check-peer-deps.ts, docs-list.ts, etc. |

## CONVENTIONS

**Language:** TypeScript (strict mode, ESM)

**Naming:**
- Convex function names: verb-first (getBySlug, publishVersion, recordDownload)
- React components: PascalCase (SkillCard, UserProfile, SearchResults)
- Utilities: camelCase (formatDate, parseSkillSlug, validateVersion)
- Constants: SCREAMING_SNAKE_CASE (MAX_UPLOAD_SIZE, DEFAULT_PAGE_SIZE)

**Code style:**
- Indentation: 2 spaces
- Quotes: single quotes (Biome)
- Lint/format: Biome + oxlint (type-aware)
- TypeScript: strict mode, no any (use unknown or proper types)

**Convex query optimization:**
- Always use .withIndex() instead of .filter() for indexed fields (avoid full table scans)
- Convex reads entire documents (no field projections), denormalize lightweight summaries for large docs
- Denormalization pattern: persist computed fields so they can be indexed, write cursor-based backfill for new fields
- Cron jobs must never scan entire tables (use indexed queries with equality filters, cursor-based pagination)
- 32K document limit per query (split .collect() calls by partition field)
- Common mistakes: .filter().collect() without index, ctx.db.get() on large docs in loop, while loops that paginate whole table

**Testing:**
- Framework: Vitest 4 + jsdom
- Tests live in src/** and convex/lib/**
- Coverage threshold: 80% global (lines/functions/branches/statements)
- Example: convex/lib/skills.test.ts

**Commit messages:**
- Conventional Commits (feat:, fix:, chore:, docs:, test:, refactor:)
- Keep changes scoped (avoid repo-wide search/replace)

**Pull requests:**
- Include summary + test commands run
- Add screenshots for UI changes
- Before merging: verify TypeScript cleanly with bunx tsc -p packages/schema/tsconfig.json --noEmit and bunx tsc -p packages/clawdhub/tsconfig.json --noEmit
- If Convex code changed: run repo typecheck path used by deploy so bunx convex deploy will not fail on tsc
- GitHub comments: for multiline gh comments/close messages, use --body-file, --input, or stdin/heredoc with real newlines (never pass literal \\n in shell strings)
- Reject PRs that add skills into source code/repo content directly (skills must be uploaded/published via CLI)

**Security:**
- Never commit secrets (.env.local)
- Convex env holds JWT keys
- Vercel only needs VITE_CONVEX_URL + VITE_CONVEX_SITE_URL
- OAuth: GitHub OAuth App credentials required for login

**Convex ops:**
- New Convex functions must be pushed before convex run (use bunx convex dev --once for dev, bunx convex deploy for prod)
- For non-interactive prod deploys: bunx convex deploy -y (skip confirmation)
- If bunx convex run --env-file .env.local returns 401 MissingAccessToken despite bunx convex login, workaround: omit --env-file and use --deployment-name <name> / --prod

**Deployment health:**
- Before writing or reviewing Convex queries, check deployment health
- Run bunx convex insights to check for OCC conflicts, bytesReadLimit, documentsReadLimit errors
- Run bunx convex logs --failure to see individual error messages and stack traces
- This helps identify which functions are causing bandwidth issues

## ANTI-PATTERNS

**Do not:**
- Use .filter() instead of .withIndex() for indexed fields (causes full table scans, massive bandwidth usage)
- Use .filter().collect() without an index (reads entire table)
- Use ctx.db.get() on large docs in a loop for list views (denormalize summaries instead)
- Use while loops that paginate the whole table to find filtered results (use indexed queries)
- Scan entire tables in cron jobs (use indexed queries with equality filters, cursor-based pagination)
- Commit secrets (.env.local, API keys, tokens)
- Add skills into source code/repo content directly (skills must be uploaded/published via CLI)
- Use literal \\n in shell strings for multiline GitHub comments (use --body-file, --input, or stdin/heredoc)
- Skip TypeScript typecheck before merging PRs (bunx tsc -p packages/schema/tsconfig.json --noEmit, bunx tsc -p packages/clawdhub/tsconfig.json --noEmit)
- Use any type (use unknown or proper types)
- Make repo-wide search/replace changes (keep changes scoped)
- Skip coverage threshold (80% global)
- Use git branch -d/-D when policy-blocked (use git update-ref -d refs/heads/<branch> instead)

## COMMANDS

```bash
# Development
bun run dev                         # local app server at http://localhost:3000
bunx convex dev                     # Convex dev deployment + function watcher
bunx convex codegen                 # regenerate convex/_generated

# Build
bun run build                       # production build (Vite + Nitro)
bun run preview                     # preview built app

# Lint/Format
bun run lint                        # Biome + oxlint (type-aware)
bun run lint:fix                    # auto-fix linting issues
bun run format                      # oxfmt --write

# Test
bun run test                        # Vitest (unit tests)
bun run test:watch                  # Vitest watch mode
bun run coverage                    # coverage run (keep global >= 80%)
bun run test:e2e                    # E2E tests (Vitest)
bun run test:e2e:local              # E2E tests (Playwright, local)
bun run test:pw                     # Playwright tests

# TypeScript
bunx tsc -p packages/schema/tsconfig.json --noEmit
bunx tsc -p packages/clawdhub/tsconfig.json --noEmit

# Convex
bunx convex deploy                  # deploy to production
bunx convex deploy -y               # deploy to production (skip confirmation)
bunx convex dev --once              # push functions once (dev)
bunx convex run <function>          # run a Convex function
bunx convex insights                # check deployment health (OCC conflicts, bandwidth)
bunx convex logs --failure          # view failed function logs

# Scripts
bun run check:peers                 # check peer dependencies
bun run docs:list                   # list documentation files
bun run verify:convex-contract      # verify Convex contract
```

## NOTES

**URL structure:**
- Canonical site: https://clawhub.ai
- Skill page: https://clawhub.ai/{owner}/{slug} (owner handle preferred, falls back to owner id)
- Skill API detail: https://clawhub.ai/api/v1/skills/{slug}
- Skill file: https://clawhub.ai/api/v1/skills/{slug}/file?path=SKILL.md

**Convex bandwidth optimization:**
- Convex reads entire documents (no field projections)
- For large docs (~6 KB+), denormalize lightweight summaries onto parent doc or use lookup table
- Examples: embeddingSkillMap, skill.latestVersionSummary, skill.badges
- Denormalization pattern: persist computed fields so they can be indexed, write cursor-based backfill for new fields
- Examples: backfillIsSuspiciousInternal, backfillLatestVersionSummaryInternal, backfillDenormalizedBadgesInternal

**Convex query limits:**
- 32K document limit per query
- Split .collect() calls by partition field (e.g., one day at a time instead of 7-day range)
- Example: rebuildTrendingLeaderboardAction in convex/leaderboards.ts

**Workspace structure:**
- Monorepo with bun workspaces
- packages/schema: shared schema types
- packages/clawdhub: shared utilities
- convex/_generated: auto-generated Convex API/types (committed for builds)

**Git notes:**
- If git branch -d/-D <branch> is policy-blocked, delete local ref directly: git update-ref -d refs/heads/<branch>

**Skill publishing:**
- Skills must be uploaded/published via CLI (not added to source code/repo content)
- Semantic versioning enforced
- Changelog tracking (auto-generated or user-provided)
- Ownership verification required for transfers

**Moderation:**
- Manual moderation overrides (staff can mark skills as clean/hidden/removed)
- Suspicious content detection (automated flagging)
- Staff audit logs (track all moderation actions)

**GitHub integration:**
- Automatic skill backups to GitHub repos
- GitHub OAuth for authentication
- GitHub profile sync (avatar, bio, created_at)

**Leaderboards:**
- Trending leaderboard (time-weighted download/star/comment scores)
- Popular leaderboard (all-time download/star/comment scores)
- Rebuilt via cron jobs (incremental updates, cursor-based pagination)

**Search:**
- TF-IDF search implementation
- Skill indexing (title, summary, description, tags)
- Search ranking (relevance score + popularity boost)
