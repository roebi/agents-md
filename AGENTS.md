# AGENTS.md - roebi shared conventions

> Canonical URL: https://raw.githubusercontent.com/roebi/agents-md/main/AGENTS.md
> Spec: https://agents.md | Steward: AAIF / Linux Foundation

This file contains the shared coding conventions for all roebi repositories.
Each project repo links here from its own local AGENTS.md.

---

## Non-negotiable coding rules (all repos)

These 5 rules apply to every task without exception:

1. No code without written requirements
2. No code without an approved plan
3. Test strategy must be defined before the first production line (TDD)
4. Testability is an architectural constraint - use DI, pure functions, side-effect isolation
5. No task is done without tests - untestable or test-free code is incomplete

---

## Text and encoding rules

- ascii-safe-text: use hyphen-minus (`-`) only - never em-dash or en-dash
- Use `->` or `<-` only - never Unicode arrow symbols
- Python source: ASCII-only OR declare `# -*- coding: utf-8 -*-` as first line

---

## Security gates (mandatory CI)

- SAST: `semgrep` or `bandit` runs on every PR - new findings fail the build
- DAST: OWASP ZAP or nuclei runs in integration pipeline against live container
- Every new endpoint triggers a full DAST re-scan
- Security scanning is an architectural constraint - design module boundaries to be statically analyzable

---

## Python tooling

- Package manager: `uv` only - never `pip` or `pipx` directly
- Lockfile: always commit `uv.lock`; use `uv lock` / `uv sync`
- Layout: src layout - `src/<module_name>/` with hatchling auto-discovery
- CI matrix: always include Python 3.14, 3.13, 3.12
- Dep policy: pin with version bounds; 3-day minimum age before adopting new releases
- Audit: `uv run pip-audit` - pin `pip-audit==2.10.0`
- Dockerfile audit pattern:
  ```
  RUN pip install --no-cache-dir "pip-audit==2.10.0" \
    && pip-audit --skip-editable \
    && pip uninstall -y pip-audit
  ```

### PyPI publishing

- Trusted Publishing (OIDC) via `pypa/gh-action-pypi-publish` v1.14.0
- SHA-pin: `cef221092ed1bacb1cc03d23a2d87d1d172e277b`
- Config: `environment: name: pypi`, `permissions: id-token: write`, `attestations: true`
- Validate `pyproject.toml` with `check_pyproject_meta.py` (18 tests) as CI step
- Every `pyproject.toml` must include: license, authors (name + email), keywords,
  classifiers (incl. Python version classifier), `[project.urls]` with
  Homepage + Repository + Issues + Changelog

### Python CLI tools

- Always use Typer - provides `--install-completion` for free
- Always demonstrate shell completion when building CLI tools

---

## Node.js tooling

- Always use `npm ci` - never `npm install`
- Always commit `package-lock.json`
- Dockerfile audit: `RUN npm audit --audit-level=moderate` after `npm ci`
- npm OIDC Trusted Publishing requires npm >= 11.5.1 and Node version 22.14.0 or higher.

---

## Agent Skills (agentskills.io)

- Skills are defined as versioned, portable `SKILL.md` files
- Skills must NEVER contain Claude Code slash commands or runtime-specific commands
- Skills must be runtime-agnostic per the agentskills.io specification: https://agentskills.io/specification
- Canonical skill library: https://github.com/roebi/agent-skills
- If a skill fits in one SKILL.md (under 500 lines) merge everything into that
  single file - no `references/` directory

---

## Secrets and infrastructure

- Secrets go in Jenkins Credentials Store - never in `.env` files committed to git
- Run services in isolated Podman - no host-mounted `~/.ssh` or `~/.gitconfig`

---

## Git and GitHub

- `tag-push-protection` Rulesets are active on all publishing repos
- Never force-push tags
- PR title format: `[<scope>] <short description>`
- Run lint and tests before every commit

---

## Output delivery

- Package multi-file outputs as `.tar.gz` archive
- Include: `.git`
- Exclude: `.venv`, `__pycache__`, `.pytest_cache`
- The archive is the canonical deliverable
