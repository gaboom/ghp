# AGENTS.md

Always-on instructions and mental model for AI agents working in `ghp` (and in projects seeded by `ghp`).

---

## 1. Current Repository Phase (Read This First)

This repository is currently in the **specification and design phase**.

* **Before making changes:** Read [SPEC.md](./SPEC.md), [TODO.md](./TODO.md), and [Q.md](./Q.md).
* **Do not add application code, scripts, workflows, or `SKILL.md` implementation files yet** unless explicitly instructed after the open questions in [Q.md](./Q.md) are resolved.
* Keep `README.md`, `SPEC.md`, `TODO.md`, `Q.md`, and `AGENTS.md` consistent whenever decisions change.

---

## 2. Core Harness Principles ("Think in GitHub Pages")

When `ghp` is active as a seed/harness for a user's site, follow these non-negotiable principles:

### A. Creative Freedom, Guided Ladder
* **Do not box the user in.** Start with the simplest static solution that fulfills their goal (plain root `index.html`, zero build required to publish).
* **Guide, never force.** Only introduce build tools, subdirectories, GitHub Actions workers, private/public repo splits, external backends, or SSO when the user's requirements genuinely call for them.

### B. Static Hosting & Cross-Origin Reality
* **GitHub Pages is static `GET`-only hosting.** There is no server runtime, server database, or secret environment at request time.
* **No cross-origin GitHub OAuth from the browser.** GitHub's OAuth token-exchange endpoint does not allow browser CORS; a pure static site cannot act as its own OAuth server.
* **Model state explicitly across layers:**
  1. **Authoring state** (files/JSON/Markdown in Git, edited by user + agent or CI).
  2. **Published read state** (static files/JSON fetched same-origin from the live site).
  3. **Visitor runtime state** (`localStorage`, `IndexedDB`, URL hash/params).
  4. **External / multi-writer state** (GitHub Issue-Ops or explicit external backends/APIs when live multi-user writes are needed).

### C. Public Repo Safety & Human-in-the-Loop Auth
* **Treat public Pages repos as completely public.** Never commit API secrets, client secrets, private keys, admin tokens, `.env` files, or unintended PII. Distinguish clearly between public client IDs and private secrets when adding 3rd-party integrations.
* **Never bypass or automate account signup, CAPTCHAs, or 2FA.** Hand off account creation and login to the user's browser (`gh auth login --web`).
* **Respect local filesystem safety.** Never run `git init` directly inside `$HOME` or `/`; place projects in a dedicated project folder.

### D. Subpath & Local Preview Discipline
* **Design for subpath hosting from Day 1.** Project sites live at `https://<user>.github.io/<repo>/` (subpath), while user sites and v3 custom domains live at `/`. Prefer relative paths (`./...`) or one central base-path config; avoid fragile root-absolute `/...` assumptions.
* **Always preview over HTTP, never `file://`.** `file://` blocks ES modules and `fetch()`. Run a local static web server for previewing, and run local builds if/when the project uses a build step.
* **Verify before and after pushing.** Validate JSON/links/secrets/file sizes before commit, and verify GitHub Actions / Pages deployment status after push.
