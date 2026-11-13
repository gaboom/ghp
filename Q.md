# Open Design Questions (`Q.md`)

A structured questionnaire to finalize decisions before implementing `ghp`. We can walk through these together section by section.

---

## Section A: `ghp` Repo Identity & Distribution

### Q1. What should `gaboom.github.io/ghp/` itself show?
* **Context:** If someone deploys `gaboom/ghp` as Pages—or if a beginner creates a repo from `ghp` and pushes before customizing—what is in the root `index.html`?
* **Options:**
  * **A.** A clean **generic starter site** (*"Your Club / Business / Topic"*) so users never accidentally publish an ad for `ghp`.
  * **B.** A **`ghp` project showcase / interactive guide** (with the agent instructed to replace it during onboarding).
* **Proposed Default:** **A** for `index.html` (generic clean seed, while `README.md` documents `ghp`), OR a tiny toggleable sample clearly marked as starter content.
* **Status:** Decided — **Decision:** **Option A** for now (clean generic starter seed). Once v3 custom domain support is introduced, `ghp` will dogfood itself to build and host its own proper homepage.

### Q2. Regular repo first vs. Template repo later—and skill updates?
* **Context:** We confirmed `gaboom/ghp` can start as a regular repo and flip on GitHub's **Template repository** setting later, and `SKILL.md` can support both (`--template` when enabled, clean shallow-copy + fresh `git init` fallback when not, plus optional `fork`).
* **Question:** Once a user creates a site repo from `ghp`, should `SKILL.md` include a **"check/sync skill updates from `gaboom/ghp`"** procedure (updating `.agents/skills/ghp/` without touching site files), or should each site stay self-contained at creation time?
* **Proposed Default:** Include an optional, manual *"update `ghp` harness"* command using a small `.ghp-version` marker, never auto-running without asking.
* **Status:** Decided — **Decision:** Needed eventually and documented as a future capability in `SPEC.md`, **not** implemented for v0/v1.

---

## Section B: Build & Worker Semantics

### Q3. Clarifying "with and without google webworker build"
* **Context:** Earlier you mentioned *"with and without google webworker build, i want to be able to build locally, BUT if needed at all"*. This could refer to **GitHub Actions workflow builds**, **browser Web Workers / Service Workers**, or both.
* **Question:** Which exact capabilities should be modeled?
  * **A.** **Zero-build publishing** (default v1) + **GitHub Actions CI workflow build** (optional v2) + **local build** (optional v2).
  * **B.** Also explicitly cover **browser Web Workers / Service Workers** (e.g., offline caching / background computation, including `/<repo>/` Service Worker scope traps).
* **Proposed Default:** Cover **both** cleanly—zero-build vs. Actions/local build in Rung 2, and browser Web Workers / Service Workers in Rung 1/2.
* **Status:** Decided — **Decision:** **Option A only** (zero-build publishing by default, plus optional GitHub Actions CI workflow builds and optional local builds). Browser Web Workers / Service Workers are not needed as a core framework concern.

### Q4. Preferred tooling when a user *does* climb to a local/CI build (Rung 2)?
* **Context:** v1 has zero build dependencies. When an advanced user (e.g., sailing club with Markdown blogs/logs) asks for a build step, the agent could pick anything (Node/Vite/Astro/11ty, Python, shell script).
* **Question:** Should the harness stay completely **tool-agnostic** (let the user/agent choose) or recommend a **default lightweight stack** when a build becomes necessary?
* **Proposed Default:** Tool-agnostic by default, with a short recommended path (e.g., Node + Vite or Eleventy/Astro) documented in `references/02-workflows-and-builds.md` for consistency.
* **Status:** Decided — **Decision:** **Strictly tool-agnostic.** Let the harness principles + user's needs guide whatever technology fits best (whether static HTML, JS frameworks like Angular, Rust, WebAssembly bytecode, etc.), while always reminding the agent of GitHub Pages' static constraints and suggested workaround patterns (`localStorage`, static JSON/Git workflows, integration hints)—deep workaround recipes are deferred beyond v0.

---

## Section C: v1 Plain Root Seed Design

### Q5. Root `index.html` structure: Single shell + hash routes vs. Multi-page HTML?
* **Context:** v1 lives at root (`./index.html`) with no build step.
* **Options:**
  * **A. Single `index.html` with `#/` hash sections/views + `data/site.json`**
  * **B. Multi-page HTML (`index.html`, `about.html`, etc.)**
  * **C. Hybrid single-page starter**
* **Status:** Decided — **Decision:** **Do not force one structure.** Keep the v0/v1 seed minimal, explain all routing/page options and their GitHub Pages trade-offs (SEO/Open Graph vs. shared state, `404.html` behavior, relative paths) to the agent, and **let the agent choose the best architecture** for the user's goal.

### Q6. How much "state management" should ship in the v0/v1 seed?
* **Context:** We want v0/v1 to feel effortless for a simple business/club/topic page, while demonstrating clean static state management.
* **Status:** Deferred to Scope Review — **Decision:** Define and review the **exact v0 feature overview & scope** together right after finishing the remaining `Q.md` questions, before any implementation starts.

### Q7. Excluding harness files when publishing from root `.`
* **Context:** Since v1 puts `index.html` at the repo root, `.agents/`, `AGENTS.md`, `SPEC.md`, `TODO.md`, `Q.md`, `README.md`, and `LICENSE` also sit at root.
* **Question:** Should `.github/workflows/deploy.yml` stage root `.` while **excluding harness/planning files**?
* **Status:** Decided — **Decision:** **Not needed for the initial POC**—keep v0 publishing as simple as possible first, then support **both** (plain root publish and clean harness-excluded / `PUBLISH_DIR` staging) in later iterations.

---

## Section D: URL-First Bootstrap & BYO Agent Portability

### Q8. Self-contained URL bootstrap mechanics (Needs Explanation -> Open)
* **Context:** Suppose a user does **not** fork or clone `gaboom/ghp` first. Instead, they open an empty folder in Antigravity and paste *only* the raw GitHub URL to `SKILL.md` (e.g. `https://raw.githubusercontent.com/gaboom/ghp/main/.agents/skills/ghp/SKILL.md`). At that moment, the agent has read `SKILL.md`, **but none of the other files in `gaboom/ghp` exist on the user's disk yet**.
* **Question:** How should `SKILL.md` bridge that gap?
  * **A. Clone/Template `gaboom/ghp`:** `SKILL.md` guides `git`/`gh` login, then creates/clones the new repo from `gaboom/ghp` (bringing in the seed + `references/` + `scripts/`), and knows the raw GitHub base URL to read any `references/*.md` before cloning if needed.
  * **B. Embed Inline Fallback Seed:** `SKILL.md` *also* contains a copy of a tiny starter `index.html` / workflow inside the Markdown itself so it can scaffold from scratch even without pulling files from `gaboom/ghp`.
* **Proposed Default:** **A** (no duplicate seed code to keep in sync; `SKILL.md` simply guides templating/cloning from `gaboom/ghp`).
* **Status:** Open — **Decision:** _TBD_

### Q9. Multi-agent discovery files
* **Context:** Antigravity discovers `.agents/skills/ghp/SKILL.md` and `AGENTS.md` (and `GEMINI.md`). Claude Code looks for `CLAUDE.md` / `.claude/skills/`. Copilot looks for `.github/copilot-instructions.md`.
* **Question:** Should we include agent-specific pointer files (`CLAUDE.md`, `GEMINI.md`, `.github/copilot-instructions.md`) in the repo root, or keep the repo agent-agnostic?
* **Status:** Decided — **Decision:** **Keep `ghp` agent-agnostic.** Stick to standard `AGENTS.md` + `.agents/skills/ghp/SKILL.md`; keep agent-specific config files out of the core repo (users/agents can add their own locally if desired).

---

## Section E: Ubuntu Zero-Knowledge UX & Safety Guardrails

### Q10. Automatic Git identity & `GH007` protection in `preflight.sh`
* **Context:** Fresh Ubuntu installs fail on `git commit` if `user.name`/`user.email` aren't set, and fail on `git push` (`GH007`) if the user's GitHub account hides their email address.
* **Question:** Should `preflight.sh` (with user confirmation) automatically run `gh auth setup-git` and configure `git config user.email` to `<id>+<login>@users.noreply.github.com` when missing?
* **Proposed Default:** **Yes** (ask before writing global git config, or set repo-local `.git/config` automatically).
* **Status:** Open — **Decision:** _TBD_

### Q11. Safe workspace location check
* **Context:** Beginners often open an agent inside `$HOME` (`/home/user`) or `~/Downloads`.
* **Question:** Should `SKILL.md` enforce a hard rule never to `git init` or clone directly into `$HOME` or `/`, and suggest `~/Sites/<project-name>` (or `~/projects/<project-name>`)?
* **Proposed Default:** **Yes** (suggest `~/Sites/<project-name>`).
* **Status:** Open — **Decision:** _TBD_

### Q12. Local preview server & browser launch behavior
* **Context:** `file://` breaks ES modules and `fetch()`. Ubuntu includes `python3` out of the box.
* **Question:** Should the preview helper pick a free port automatically, launch `python3 -m http.server`, and run `xdg-open` (or just print the `http://localhost:<port>` link for the user to click)?
* **Proposed Default:** Pick a free port (trying `8000` first), print the clickable URL clearly, and optionally open via `xdg-open` if running in a desktop session.
* **Status:** Open — **Decision:** _TBD_

### Q13. Media & binary size guardrails
* **Context:** Phone photos/videos easily exceed healthy repo sizes or GitHub's 100 MB file limit.
* **Question:** Should `check-site.sh` warn on images `> 1–2 MB`, hard-block commits `> 25 MB` (configurable), and instruct the agent to offer local compression or external video embedding?
* **Proposed Default:** **Yes** (warn > 2 MB, block > 25 MB with override guidance).
* **Status:** Open — **Decision:** _TBD_

---

## Section F: Advanced Rungs (Sailing Club, SSO, Custom DNS & HTTPS)

### Q14. Recommended progression for multi-user dynamic club data
* **Context:** A sailing club with events, blogs, and sailing logs may outgrow single-author Git edits.
* **Question:** In `references/04-integrations-and-sso.md`, do you agree with steering users in order:
  1. Single maintainer + agent in Git (`data/*.json` / Markdown),
  2. Club members submitting via GitHub Issue forms + Actions (zero external backend),
  3. External BaaS / custom backend + 3rd-party SSO + `<iframe>` / Web Components?
* **Proposed Default:** **Yes**, framed as options (guided, never forced).
* **Status:** Open — **Decision:** _TBD_

### Q15. v3 Custom Domain (DNS & HTTPS) scope
* **Context:** Setting up DNS and HTTPS on GitHub Pages has well-known registrar traps (apex `A`/`AAAA` vs. `www` `CNAME`, Cloudflare orange-cloud proxy blocking initial GitHub TLS cert issuance, subpath `/` flip).
* **Question:** Is it enough for v3 to provide a registrar-agnostic `05-dns-and-https.md` playbook + `dig` / `curl` verification commands in the skill, or do you also want a script/check that verifies DNS propagation and HTTPS cert readiness?
* **Proposed Default:** Playbook + `dig` / `gh api` Pages status checks in the skill (no heavy external dependencies).
* **Status:** Open — **Decision:** _TBD_
