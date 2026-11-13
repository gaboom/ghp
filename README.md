# ghp

**GitHub Pages makes a good homepage.**

`ghp` is a repository seed and cognitive agent harness for creating, running, and scaling websites on **GitHub Pages**. You bring your own AI agent (such as **Google Antigravity**, Claude Code, Codex, or Cursor); `ghp` supplies the instructions, guardrails, and starter seed so your agent knows how to **think in GitHub Pages**—without locking you into a rigid framework.

> **Project Status:** Specification & design phase (`SPEC.md`, `TODO.md`, `Q.md`). Implementation files will be added once open design questions in [Q.md](./Q.md) are resolved.

---

## Philosophy

* **Seed + Harness, Not a Box:** Start with a plain, zero-build root `index.html` and let your agent grow your site into anything you imagine.
* **Zero Dev Knowledge Required for v1:** Built around a simple baseline—Ubuntu, a web browser, `git`, `gh` CLI, and your agent—guiding everything from creating a GitHub account to verifying your live site.
* **Guided Complexity Ladder (Never Forced):** Stay simple forever if a static club/business/topic site is all you need, or climb step-by-step into client state, subdirectories, local/CI builds, private-source-to-public-Pages pipelines, 3rd-party `<iframe>`/Web Component integrations, custom backends, SSO, and custom DNS/HTTPS.

---

## Planned Entry Paths

1. **Repo-First (Template or Fork):** Create your own repository from `gaboom/ghp`, open it with your agent, and `AGENTS.md` + `.agents/skills/ghp/SKILL.md` load automatically.
2. **URL-First (Start from Scratch):** Point your agent to the raw `SKILL.md` URL in an empty workspace and let it walk you through tool checks, GitHub login, repo creation, local preview, and publishing.

---

## Project Documents

| Document | Purpose |
| :--- | :--- |
| [SPEC.md](./SPEC.md) | Full architecture, constraints, state model, onboarding flow, and Complexity Ladder |
| [AGENTS.md](./AGENTS.md) | Always-on rules and "Think in GitHub Pages" principles for AI agents |
| [TODO.md](./TODO.md) | Phased implementation roadmap |
| [Q.md](./Q.md) | Open design questions and decisions to review together |
| [LICENSE](./LICENSE) | CC0 1.0 Universal (Public Domain Dedication) |
