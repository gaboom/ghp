# ghp Specification (`SPEC.md`)

> **Status:** Draft v0.1 (Pre-implementation — captures design architecture and requirements prior to resolving [Q.md](./Q.md)).

---

## 1. Vision & Philosophy

**`ghp`** ("GitHub Pages makes a good homepage") is a **repository seed and self-contained AI agent harness**—not a locked-in CMS or heavy framework.

* **Bring Your Own Agent (BYO Agent):** Users bring their preferred agent (starting with **Google Antigravity**, and portable to Claude Code, Codex, Cursor, etc.). `ghp` provides the structured knowledge (`SKILL.md`, `AGENTS.md`, `references/`, `scripts/`) that teaches the agent how to **think in GitHub Pages**.
* **Seed, Not Cage:** Day 1 starts with a clean, plain root `index.html` static site that requires **zero build step** to publish. As creative ambition grows, the harness guides the user up a **Complexity Ladder** without forcing any complexity upfront.
* **Dual Entry:**
  1. **Repo-First:** Use `gaboom/ghp` as a template or fork, open it with an agent, and go.
  2. **URL-First:** Load `SKILL.md` via its GitHub URL into an agent and let it guide everything from checking `git`/`gh` and creating a GitHub account to initializing, running locally, and publishing the site.

---

## 2. Target Personas & Use Cases

### 2.1 Basic Use Case (v1 Baseline)
* **User:** Zero development knowledge.
* **Environment:** Ubuntu Linux desktop, web browser, `git` + `gh` CLI (installed or guided through installation), Google Antigravity.
* **Goal:** Create and maintain a **fully static website** on a **public GitHub repository** published via **GitHub Pages** for a business, club, or topic page.
* **Expectation:** Plain-language conversation; safe local preview via HTTP server; push-to-publish with no manual Git gymnastics or build toolchain required.

### 2.2 Advanced Use Case (Complexity Ladder Target)
* **User:** Advanced site owner or club maintainer (e.g., a **sailing club**).
* **Capabilities Needed Over Time:**
  * Rich structured content: events calendar, blogs, sailing logs, member resources.
  * Subdirectory layout (`site/`, `docs/`, `dist/`) when outgrowing root-level files.
  * Optional **local build** and/or **GitHub Actions CI workflow build** (strictly tool-agnostic: whatever the user and agent choose—e.g., static generators, Angular, Rust/WebAssembly, asset optimization), while keeping buildless publishing supported for simple sites.
  * Browser-side interactivity (client stores, URL state).
  * **3rd-party integrations:** `<iframe>` embeds, custom Web Components, external APIs.
  * **Custom backends & 3rd-party SSO:** Authentication and live multi-user data integrated cleanly across static hosting boundaries.
  * **Private source → Public Pages split:** Keeping drafts, raw data, or source automation in a private repository while publishing static artifacts to a public GitHub Pages repository.
  * **Custom domain (v3):** DNS (`A`/`AAAA`/`CNAME`), `CNAME` file, and HTTPS enforcement.

---

## 3. Platform Constraints & Realities

`ghp` designs *around* GitHub and GitHub Pages constraints rather than hiding them until things break:

1. **Free Tier Requires Public Repos for Pages:**
   * GitHub Pages on free personal accounts requires a **public** repository (private Pages require paid plans).
   * Consequence: Public Pages repos must **never** contain secrets, private tokens, or unintended PII. When private source/drafts are needed on a free account, use the **Private Source → Public Pages** split (Rung 3).
2. **Static Hosting Only (`GET`-Only):**
   * GitHub Pages executes no server-side code, stores no server sessions, and provides no server rewrite rules (`.htaccess` / `_redirects`).
   * Unknown paths serve `404.html`; multi-view routing must use real HTML files, hash routing (`#/route`), or a `404.html` redirect pattern.
3. **Cross-Origin (CORS) Constraint for GitHub Auth:**
   * GitHub's browser OAuth token-exchange endpoint does not support cross-origin (`CORS`) requests from a static `github.io` origin.
   * Consequence: A pure static site cannot act as an independent GitHub OAuth app without an external backend/proxy, and authoring writes belong with the **local user + agent** (via `git`/`gh`) or **GitHub Actions** rather than an unproxied browser admin panel.
4. **Subpath Hosting Gotcha (`/<repo>/` vs. `/`):**
   * Project Pages serve from `https://<username>.github.io/<repo-name>/`, whereas user sites (`<username>.github.io`) and v3 custom domains serve from `/`.
   * Consequence: Hardcoded root-absolute paths (`/styles.css`, `/data/site.json`) break immediately on project Pages and again during custom-domain migration. **Relative paths (`./...`) or a single configurable base path** are mandatory from Day 1.
5. **Quotas & Media Limits:**
   * GitHub enforces a 100 MB hard per-file limit (50 MB warning) and ~1 GB soft repository/site guidelines. Uncompressed photos/videos from phones are the primary failure mode for non-developers.

---

## 4. Three-Tier State Management Model

To enable "complex static homepage systems with state management" without a backend required on Day 1, state is cleanly separated by ownership and lifetime:

| Layer | Where It Lives | Who Writes It | How Site Reads It |
| :--- | :--- | :--- | :--- |
| **1. Authoring / CMS State** | `data/*.json`, Markdown, config in Git | User + BYO Agent locally, or GitHub Actions (cron / Issue-Ops) | Baked into HTML or fetched same-origin (`./data/...`) |
| **2. Published Read State** | Static JSON/assets deployed on GitHub Pages | Build / Deploy pipeline | Same-origin `fetch()` (zero CORS, CDN-cached, no API rate limit) |
| **3. Visitor Runtime State** | `localStorage`, `IndexedDB`, URL hash/query params, JS store | Visitor's browser | Immediate in-memory / client storage |
| **4. Multi-Writer / Live State** *(Advanced)* | GitHub Issues/Discussions bridge or External BaaS / Custom API | Authenticated visitors / 3rd-party services | REST/GraphQL/WebSocket fetch to CORS-enabled backend |

---

## 5. Onboarding, Preflight & Repo Creation Architecture

### 5.1 Ubuntu Preflight & Validation
Before writing site files, the skill verifies the user's environment:
* Check availability of `git` and `gh` (guide `sudo apt install git gh` on Ubuntu if missing).
* Check `gh auth status`.
  * If user lacks a GitHub account: pause and guide browser signup at `https://github.com/signup` + 2FA setup (never automate signup/CAPTCHA/2FA).
  * Authenticate via `gh auth login --web` (browser device-code flow; no raw passwords or manual PAT copy-pasting) and configure `gh auth setup-git`.
* Validate Git identity (`user.name` / `user.email`) to prevent commit failures and `GH007` private-email push rejections (preferring GitHub's `noreply` email when appropriate).
* Enforce safe workspace placement (refuse to initialize inside `$HOME` or `/`).
* Validate checkout state, remote (`origin`), clean working tree, and local HTTP server availability (`python3 -m http.server` standard on Ubuntu).

### 5.2 Supporting Both Regular Repos and Template Repos
`gaboom/ghp` may start as a regular repository and later enable GitHub's **Template repository** flag (or vice versa). The onboarding flow supports all paths transparently:
1. **Template Repo Path (`is_template == true`):** `gh repo create <name> --template gaboom/ghp --public --clone`.
2. **Regular Repo Fallback (`is_template == false`):** Create `<name>` via `gh repo create`, pull the clean seed files from `gaboom/ghp` (without copying `gaboom/ghp`'s `.git` history), initialize a fresh commit, and push.
3. **Fork Path (if explicitly desired):** `gh repo fork gaboom/ghp --clone`.

---

## 6. The Complexity Ladder (`v1` → `v2` → `v3`)

The harness organizes capabilities into progressive rungs. Agents only load deep reference playbooks when the user asks for that rung:

* **v1 — Rung 0 & 1: Plain Root Static Site (Zero Build)**
  * Root `index.html` (plus optional assets/data/pages chosen by the agent based on user goals and Pages routing trade-offs—hash vs. multi-page HTML vs. `404.html`), starting with clean generic starter content so new repos never accidentally publish a `ghp` ad; `ghp` will dogfood itself for its own proper domain site in v3.
  * Publishes directly to GitHub Pages with **no build step required** (v0 POC uses simple root publishing; later versions support both plain root publish and clean harness-excluded / `PUBLISH_DIR` staging).
  * Runs locally via static HTTP server (`python3 -m http.server`).
  * Client-side state via JSON + `localStorage` / URL state.

* **v2 — Rung 2: Subdirectories, Local/CI Builds & GitHub Workers**
  * Graduate cleanly from root `./` to a subdirectory (`site/`, `src/`, `dist/`) via `PUBLISH_DIR`.
  * Optional **local build** and/or **GitHub Actions workflow build** (tool-agnostic: JS/TS frameworks like Angular, Rust/WASM, Python generators, etc.—always constrained by GitHub Pages' static output reality and guided toward static/client/`localStorage`/integration workarounds; deep build/integration recipes expand post-v0).
  * Automated GitHub Actions workers: scheduled cron data refreshes, Issue-Ops (turning structured GitHub Issues into PRs/JSON).

* **v2+ — Rung 3: Private Source Repo → Public Pages Repo**
  * Keep private drafts, raw member/club data, or build scripts in a **private** repo (free on GitHub) and deploy only compiled public static files to a **public** GitHub Pages repo via Actions + scoped deploy key/token.

* **v2+ — Rung 4: 3rd-Party Integrations, Custom Backends & SSO**
  * Safe embedding of `<iframe>` widgets and custom Web Components (`<custom-element>`).
  * Connecting custom backends / BaaS (Supabase, PocketBase, Cloudflare Workers, etc.) for live multi-user writes (sailing logs, RSVPs).
  * Integrating 3rd-party SSO/OAuth cleanly without leaking client secrets into the public repo.

* **v3 — Rung 5: Custom Domain (DNS & HTTPS)**
  * Configuring `CNAME` in the repo, apex (`A`/`AAAA`) or subdomain (`CNAME`) DNS records, propagation verification, Cloudflare/proxy caveats during TLS issuance, and enabling GitHub Pages **Enforce HTTPS**.
  * Smooth transition from `/<repo>/` subpath to `/` root.

---

## 7. Planned Repository Layout & Skill Packaging

```text
ghp/
├── README.md                          # Human overview + URL-first / Repo-first instructions
├── AGENTS.md                          # Always-on harness rules
├── SPEC.md                            # This architecture & requirements document
├── TODO.md                            # Phased implementation roadmap
├── Q.md                               # Open design questions for alignment
├── LICENSE                            # CC0 1.0 Universal
│
├── .agents/skills/ghp/
│   ├── SKILL.md                       # Self-contained bootstrap + ladder router (URL-loadable)
│   ├── scripts/                       # Preflight, local serve helper, pre-publish site checks
│   └── references/                    # Progressive disclosure playbooks (Rungs 1–5)
│
├── .github/workflows/
│   └── deploy.yml                     # Stages root cleanly (no harness leak) or subdir/build
│
└── [v1 root static seed]              # index.html, 404.html, styles.css, app.js, data/site.json
```

*Note: Exact implementation file details remain subject to resolving [Q.md](./Q.md). A future harness/skill self-update mechanism (syncing `.agents/skills/ghp/` updates from `gaboom/ghp` without overwriting site files) is planned post-v0/v1.*

---

## 8. Safety Rails & Pre-Publish Checks

Every publish cycle should validate:
1. **Valid Data:** JSON/config files parse cleanly.
2. **Secret & PII Hygiene:** Scan staged diffs for accidental tokens, API secrets, private keys, or `.env` files.
3. **Asset Size Guard:** Warn on oversized images (`>1–2 MB`) and block files approaching GitHub's 100 MB limit.
4. **Relative Path Sanity:** Catch accidental root-absolute links that would break under `/<repo>/`.
5. **Deploy Verification:** Confirm the GitHub Actions Pages workflow succeeds and return the live URL.
