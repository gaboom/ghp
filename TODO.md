# TODO.md

Phased implementation plan for `ghp`. **Do not begin Phase 1+ code/implementation until Phase 0 ([Q.md](./Q.md)) is reviewed and resolved.**

---

## Phase 0: Design Alignment (Current)
- [x] Initialize core planning docs (`README.md`, `AGENTS.md`, `SPEC.md`, `TODO.md`, `Q.md`).
- [ ] Review and resolve all open design questions in [Q.md](./Q.md).
- [ ] Define and approve the explicit **v0 POC scope & feature overview** before starting implementation.
- [ ] Update `SPEC.md` and `AGENTS.md` with finalized decisions from `Q.md`.

---

## Phase 1: Agent Harness & Onboarding Tooling
- [ ] Author `.agents/skills/ghp/SKILL.md`:
  - [ ] Self-contained URL-first bootstrap instructions (works before repo is cloned).
  - [ ] Repo-first flow supporting both `is_template == true`, regular-repo clean copy fallback, and optional fork.
  - [ ] Step-by-step Ubuntu onboarding (`git`, `gh`, browser signup + 2FA handoff, `gh auth login --web`, `gh auth setup-git`).
  - [ ] Complexity Ladder decision map linking to `references/`.
- [ ] Keep harness agent-agnostic (`AGENTS.md` + `.agents/skills/ghp/SKILL.md` only; no vendor-specific root pointer files).
- [ ] Create `.agents/skills/ghp/scripts/preflight.sh` (tool checks, auth status, safe workspace check, git identity/noreply check, remote/checkout validation).
- [ ] Create `.agents/skills/ghp/scripts/check-site.sh` (JSON validation, secret/PII scan, binary size guard, subpath link check).
- [ ] Create local preview helper (or document reliable `python3 -m http.server` dynamic-port wrapper).

---

## Phase 2: v1 Plain Root Seed & Zero-Build Publishing
- [ ] Create minimal plain root seed (`index.html`, `404.html`, `styles.css`, `app.js`, `data/site.json`) using relative paths and CSS custom properties.
- [ ] Ensure v1 seed demonstrates clean declarative state (`data/site.json`) + optional client runtime state (`localStorage` / URL state) without requiring any build tool.
- [ ] Create `.github/workflows/deploy.yml` (simple root publish for v0 POC; later support both plain root publish and harness-excluded / `PUBLISH_DIR` staging + build hooks).
- [ ] Update `README.md` with exact quickstart prompts, raw `SKILL.md` URL, and human instructions.

---

## Phase 3: Complexity Ladder Playbooks (`references/`)
- [ ] `references/01-static-and-state.md` — Subpath safety, multi-page vs. hash routing, `data/*.json`, client state, media optimization.
- [ ] `references/02-workflows-and-builds.md` — Migrating root `./` → subdirectory (`site/`, `dist/`), tool-agnostic local builds, GitHub Actions CI builds, cron fetchers, Issue-Ops workflows.
- [ ] `references/03-private-to-public.md` — Private source repo publishing static output to a separate public GitHub Pages repo.
- [ ] `references/04-integrations-and-sso.md` — Advanced club/app integrations: `<iframe>`, Web Components, custom backends, CORS boundaries, 3rd-party SSO/OAuth without leaking secrets.
- [ ] `references/05-dns-and-https.md` — v3 custom domain setup (`CNAME`, apex `A`/`AAAA` vs. `www` `CNAME`, Cloudflare proxy caveat, GitHub Pages HTTPS enforcement, `/` migration check).

---

## Phase 4: Verification & Release
- [ ] Test **Repo-first** path on clean Ubuntu environment (preflight → customize → local HTTP serve → publish → verify live Pages URL).
- [ ] Test **URL-first** path in an empty workspace using only the raw GitHub `SKILL.md` URL.
- [ ] Test **Regular-repo fallback** vs. **Template-repo (`--template`)** creation paths.
- [ ] Test **v2 migration** (root → subdirectory + optional build) on a sample sailing club scenario.
- [ ] Optionally enable **Template repository** in GitHub Settings for `gaboom/ghp` when ready for public launch.
- [ ] *(Post-v0/v1)* Design and implement skill/harness update sync from `gaboom/ghp` without overwriting user site files.
- [ ] *(v3 Dogfooding)* Build and host `ghp`’s own proper homepage using `ghp` + custom domain support.
