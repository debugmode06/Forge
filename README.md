# 🔨 Forge

**The AI-Native Code Collaboration Platform**

> Replace GitHub with something smarter. Forge solves the three deepest problems GitHub leaves unsolved — with AI that understands intent, not just diffs.

[![License: MIT](https://img.shields.io/badge/License-MIT-7C3AED.svg)](https://opensource.org/licenses/MIT)
[![Stack: Next.js 14](https://img.shields.io/badge/Next.js-14-black)](https://nextjs.org)
[![DB: Supabase](https://img.shields.io/badge/DB-Supabase-3ECF8E)](https://supabase.com)
[![AI: Gemini 1.5 Flash](https://img.shields.io/badge/AI-Gemini%201.5%20Flash-4285F4)](https://aistudio.google.com)
[![Cost: $0/month](https://img.shields.io/badge/Launch%20Cost-%240%2Fmonth-3FB950)](https://github.com)

---

## What is Forge?

Forge is an open-core, AI-native code collaboration platform built for developer teams who want more than version control. It combines a full GitHub-alternative git workflow with AI features that no existing platform offers.

**Three core problems Forge solves:**

| Problem | GitHub Today | Forge Solution |
|---|---|---|
| Merge conflicts | Shows conflicting lines, forces manual decision with no context | AI reads the intent of both branches and proposes a merged version with explanation |
| Code knowledge | No way to ask "why was this written?" | Knowledge graph traces every line to its PR, author, and ticket |
| PR review load | Human reviewer reads diffs cold | AI pre-reviews with risk score, test gap analysis, and suggestions before a human sees it |

---

## Features

### AI-Powered
- **Smart Conflict Resolver** — Gemini 1.5 Flash understands intent, proposes resolutions with confidence scores
- **Semantic Code Search** — Natural language search across all repos simultaneously (Cmd+K)
- **AI PR Pre-Review** — Risk scoring (0–100), test coverage gap detection, suggested reviewers
- **"Why was this written?"** — Hover any function to see its PR, author, and linked ticket
- **CI Log Summarizer** — AI summary card pinned above raw build logs
- **Stale Branch Detection** — AI detects abandoned work and suggests merge or archive

### Collaboration
- Pull Requests with inline code comments and threaded replies
- Real-time PR comments via Supabase Realtime WebSockets
- Live presence indicators — see who's viewing a PR right now
- AI-suggested reviewer assignment based on expertise map
- Code reviews with Approve / Request Changes flow

### Security & Access Control
- **RBAC** — Owner, Admin, Maintainer, Contributor, Viewer roles
- **JWT** (RS256, 15-min) + Refresh tokens (7-day, httpOnly, rotated on use)
- **SSH key authentication** for all git operations
- **Branch protection rules** — require PR review, prevent force push
- **HMAC-SHA256** webhook signature verification
- **TOTP 2FA** — optional per user, OTP-app compatible
- **Audit logs** — every push, merge, and access event logged with actor + IP

### Team & Organizations
- Create organizations, invite members via email
- Secure invitation tokens (expire in 24 hours)
- Per-repo permission overrides
- Role manager UI with drag-to-assign

### Developer Wellness
- Velocity dashboard — flow hours, review load, PR cycle time
- Burnout signal alerts based on commit and review patterns

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 14, TypeScript, Tailwind CSS, shadcn/ui, Framer Motion |
| Backend | Node.js, tRPC, Zod, Prisma ORM |
| Database | Supabase (Postgres + pgvector + Auth + Realtime) |
| Cache | Upstash Redis (rate limiting, sessions, pub/sub) |
| Storage | Cloudflare R2 (zero egress fees) |
| Git Backend | Gitea (self-hosted, SSH + HTTPS) |
| CI/CD | Drone CI (self-hosted) |
| AI — Conflicts | Google Gemini 1.5 Flash (1500 req/day free) |
| AI — Embeddings | Ollama + nomic-embed-text (100% free, local) |
| AI — Classification | Groq Llama 3.1 70B (free tier) |
| Hosting | Vercel (frontend) + Railway (API) + Fly.io (git server) |

**Launch cost: $0/month** — entirely within free tiers up to ~100 users.

---

## Getting Started

### Prerequisites

- Node.js 20+
- pnpm 9+
- Docker Desktop
- [Ollama](https://ollama.com) installed locally
- [Supabase CLI](https://supabase.com/docs/guides/cli)
- [Fly CLI](https://fly.io/docs/hands-on/install-flyctl/)

### Local Setup

```bash
# 1. Clone and install
git clone https://github.com/yourname/forge.git
cd forge
pnpm install

# 2. Start local Supabase (Postgres + pgvector + Auth + Realtime)
supabase start

# 3. Pull and run Ollama embedding model
ollama pull nomic-embed-text
ollama serve &

# 4. Start Gitea + Drone CI
docker-compose -f infrastructure/docker/gitea.yml up -d
docker-compose -f infrastructure/docker/drone.yml up -d

# 5. Set up environment variables
cp .env.example .env.local
# Fill in: GEMINI_API_KEY, GROQ_API_KEY, UPSTASH_REDIS_URL, R2 keys

# 6. Run migrations and seed
pnpm db:migrate
pnpm db:seed

# 7. Start all apps in parallel
pnpm dev
```

| Service | URL |
|---|---|
| Frontend | http://localhost:3000 |
| API | http://localhost:3001 |
| Gitea | http://localhost:3002 |
| Supabase Studio | http://localhost:54323 |

---

## Project Structure

```
forge/
├── apps/
│   ├── web/          # Next.js 14 frontend
│   ├── api/          # Node.js tRPC server
│   ├── ai-service/   # AI microservice (conflicts, review, embeddings)
│   └── git-server/   # Gitea webhook handler + branch protection
├── packages/
│   ├── db/           # Prisma schema + Supabase client
│   ├── ui/           # Shared shadcn/ui components
│   ├── types/        # Shared TypeScript types
│   └── config/       # ESLint, TypeScript, Tailwind configs
└── infrastructure/
    ├── fly.toml
    ├── railway.json
    └── docker/
```

---

## Deployment

### Frontend → Vercel
```bash
# Connect repo to Vercel — auto-detects Next.js
# Add NEXT_PUBLIC_* env vars in Vercel dashboard
# Set custom domain — free SSL auto-provisioned
```

### Backend API → Railway
```bash
npm install -g @railway/cli
railway login && railway init
railway variables set GEMINI_API_KEY=xxx SUPABASE_URL=xxx ...
railway up
```

### Git Server → Fly.io
```bash
flyctl auth login
flyctl launch --name forge-git
flyctl volumes create gitea_data --size 3
flyctl deploy
```

### Database → Supabase
```bash
supabase db push --linked
# Enable pgvector: Extensions > vector in Supabase dashboard
# Run HNSW index migrations in SQL editor
```

---

## RBAC Permission Matrix

| Role | View | Clone | Push | Merge PR | Manage PRs | Invite | Admin |
|---|---|---|---|---|---|---|---|
| Owner | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Admin | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | limited |
| Maintainer | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ |
| Contributor | ✓ | ✓ | ✓ | ✗ | ✓ | ✗ | ✗ |
| Viewer | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ |

---

## Roadmap

- [x] Phase 1 — Core git platform (Weeks 1–6)
- [ ] Phase 2 — AI conflict resolver (Weeks 7–12)
- [ ] Phase 3 — Knowledge graph + real-time collaboration (Weeks 13–18)
- [ ] Phase 4 — Power features: PR review, velocity, expertise map (Weeks 19–22)
- [ ] Phase 5 — Polish + Product Hunt launch (Weeks 23–24)

---

## Pricing (Post-Launch)

| Plan | Price | Features |
|---|---|---|
| Free | $0 | Unlimited public repos, 3 private repos, 1 org |
| Pro | $9/user/mo | Unlimited private repos, full AI features |
| Team | $19/user/mo | Analytics, SAML SSO, audit log export |
| Enterprise | Custom | Self-hosted, SLA, compliance exports |

> **Breakeven:** 3 Team-plan users ($57/mo) covers full infrastructure at 1,000-user scale.

---

## Security

- All routes protected via Supabase Auth JWT middleware
- Row Level Security (RLS) on every Supabase table
- Code sent to Gemini API: only conflict blocks, never full repository
- Embeddings generated locally via Ollama — no code leaves the server
- Users can opt out of AI features per repository in settings

See [SECURITY.md](./SECURITY.md) for full disclosure policy.

---

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](./CONTRIBUTING.md) first.

```bash
# Fork → feature branch → PR to main
git checkout -b feature/your-feature-name
git commit -m "feat: your feature description"
git push origin feature/your-feature-name
# Open a PR on GitHub
```

---

## License

MIT License for the open core. The AI layer is proprietary.  
See [LICENSE](./LICENSE) for details.

---

<p align="center">
  Built with <strong>Forge</strong> — The future of code collaboration
</p>
