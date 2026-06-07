# Claude Code Enterprise Security Framework

Enterprise governance framework for securing Claude Code deployments. Provides layered security controls that wrap around [Project CodeGuard](https://project-codeguard.org/) technical rules and align with OWASP GenAI security guidance.

## What This Is

A set of CLAUDE.md files and supporting documents that enforce security policies when teams use Claude Code for AI-assisted development. The framework operates at three layers:

1. **Platform enforcement** (`managed-settings-template.jsonc`): Controls what Claude Code CAN DO (tool permissions, file access restrictions). Deployed via MDM. Users cannot override.
2. **Technical code security** (Project CodeGuard plugin): Controls HOW code is generated (crypto, input validation, auth patterns, API security, container hardening). Open-source rules from CoSAI/OWASP.
3. **Enterprise governance** (CLAUDE.md files): Controls organizational policy (company accounts, secrets management tools, license compliance, AI agent risk controls).

## File Structure

```
├── CLAUDE.md                          # Root: Universal rules (loads every interaction)
├── mobile/CLAUDE.md                   # Mobile platform rules (loads in mobile directories)
├── infra/CLAUDE.md                    # Cloud, container, K8s, CI/CD rules (loads in infra directories)
├── docs/security-patterns/
│   ├── secrets-management.md          # WRONG/CORRECT code examples (loaded on demand)
│   └── agentic-security.md            # OWASP LLM Top 10 + Agentic Top 10 mapping
├── scripts/security-checks.md         # Pre-commit, SBOM, scanning commands
└── managed-settings-template.jsonc    # Platform-level permission enforcement template
```

## Quick Start (Managed / Enterprise)

For organizations with MDM (Jamf, Intune, etc.) that can deploy system-level configuration:

1. Clone this repo
2. Customize `[COMPANY NAME]` placeholder in `CLAUDE.md`
3. Install Project CodeGuard: `/plugin marketplace add cosai-oasis/project-codeguard`
4. Deploy `managed-settings-template.jsonc` via MDM (remove `_comment`, `_paths`, and `_customize` keys first)
   - macOS: `/Library/Application Support/ClaudeCode/managed-settings.json`
   - Linux: `/etc/claude-code/managed-settings.json`
   - Windows: `C:\Program Files\ClaudeCode\managed-settings.json`
5. Copy the CLAUDE.md structure into your project repositories

Managed settings cannot be overridden by users. This is the recommended approach for enterprises.

## Quick Start (Non-Managed / Individual / Small Team)

For individual developers, small teams, or environments without MDM:

### 1. Copy CLAUDE.md files into your project

```bash
# From the root of your project repository
cp -r /path/to/Claude-Security-Instructions/CLAUDE.md .
cp -r /path/to/Claude-Security-Instructions/mobile/ ./mobile/       # if building mobile apps
cp -r /path/to/Claude-Security-Instructions/infra/ ./infra/         # if writing infrastructure code
cp -r /path/to/Claude-Security-Instructions/docs/ ./docs/           # security patterns reference
```

Customize `[COMPANY NAME]` to your team or project name. Remove sections that don't apply (e.g., "Company Device and Account Requirements" for personal projects).

### 2. Install Project CodeGuard

In Claude Code, run:

```
/plugin marketplace add cosai-oasis/project-codeguard
/plugin install codeguard-security@project-codeguard
```

### 3. Configure permission rules

Without MDM, use **project-level** or **user-level** settings files to enforce permission rules.

**Project-level** (`.claude/settings.json` — commit to git, shared with collaborators):

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Bash(git push --force:*)",
      "Read(**/.env)",
      "Read(**/*.pem)"
    ],
    "ask": [
      "Bash(curl:*)",
      "Bash(git push:*)"
    ]
  }
}
```

The example above is a starter set. For the full enterprise rule list — network exfiltration tooling (`nc`, `ncat`, `telnet`, `nslookup`, `dig`, `base64`), additional secret-file globs, container/IaC `ask` rules — copy from `managed-settings-template.jsonc` and adapt. The same `permissions` schema works in `.claude/settings.json`.

**User-level** (`~/.claude/settings.json` — applies to all your projects, not shared):

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Read(**/.env)",
      "Read(**/*.pem)",
      "Read(**/*.key)"
    ]
  }
}
```

### Settings precedence (highest to lowest)

| Priority | File | Scope | Shared via git? |
|---|---|---|---|
| 1 | `managed-settings.json` (system path) | All users on machine | No (deployed via MDM) |
| 2 | `.claude/settings.local.json` | This project, this user only | No (gitignored) |
| 3 | `.claude/settings.json` | This project, all collaborators | Yes |
| 4 | `~/.claude/settings.json` | All projects for this user | No |

Rules are evaluated in order: **deny > ask > allow** (first match wins).

### Key differences from managed deployment

| Capability | Managed (MDM) | Non-Managed |
|---|---|---|
| Permission rules (deny/ask/allow) | Yes | Yes |
| Users cannot override rules | Yes | No — users can modify their own settings |
| `disableBypassPermissionsMode` | Yes | Not available |
| `allowManagedPermissionRulesOnly` | Yes | Not available |
| `strictKnownMarketplaces` | Yes | Not available |
| CLAUDE.md governance rules | Yes | Yes |
| Project CodeGuard plugin | Yes | Yes |

In non-managed environments, permission rules are **advisory** — a developer can modify their local settings. Security enforcement relies on:

- **CLAUDE.md rules** (Claude Code follows these as instructions during code generation)
- **Project-level `.claude/settings.json`** (shared with the team via git)
- **Code review** (catches anything that slips through)
- **CI/CD security scanning** (SAST, SCA, secret scanning — see `scripts/security-checks.md`)

### 4. Harden CI/CD and repository settings (strongly recommended)

Without managed-settings.json, CI/CD and repository controls become your **primary hard enforcement layer** — the only controls a developer or prompt injection cannot bypass locally.

#### Branch protection rules

Configure on your main/production branches (GitHub Settings > Branches > Branch protection):

- Require pull request reviews before merging (at least 1 reviewer)
- Require status checks to pass (wire up the CI jobs below)
- Do not allow bypassing the above settings
- Require signed commits (optional but recommended)

#### CODEOWNERS for security-sensitive files

Create `.github/CODEOWNERS` to require specific reviewers for governance files:

```
# Security-sensitive files require team lead / security review
CLAUDE.md                     @your-team-lead
**/CLAUDE.md                  @your-team-lead
.claude/                      @your-team-lead
.github/workflows/            @your-team-lead
.github/CODEOWNERS            @your-team-lead
Dockerfile                    @your-team-lead
*.tf                          @your-team-lead
```

#### CI/CD security scanning

Add these jobs to `.github/workflows/security.yml` (or equivalent). These are the **hard controls** that catch issues regardless of local settings:

```yaml
name: Security checks
on: [pull_request]

jobs:
  secret-scanning:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4  # pin by SHA in production
        with:
          fetch-depth: 0
      - name: Scan for secrets
        run: |
          pip install gitleaks || true
          gitleaks detect --source . --report-path gitleaks-report.json --exit-code 1

  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Semgrep OWASP rules
        run: |
          pip install semgrep
          semgrep --config=p/owasp-top-ten --error .

  dependency-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Audit dependencies
        run: npm audit --audit-level=high  # adjust for your package manager

  governance-file-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Flag governance file changes
        run: |
          # Warn if security-sensitive files were modified
          CHANGED=$(git diff --name-only origin/main...HEAD | grep -E '(CLAUDE\.md|\.claude/|\.github/workflows/|\.github/CODEOWNERS|Dockerfile|\.tf$)' || true)
          if [ -n "$CHANGED" ]; then
            echo "::warning::Security-sensitive files modified — require security review:"
            echo "$CHANGED"
          fi
```

#### Pre-commit hooks (local enforcement)

For developers who want local checks before commits reach CI:

```bash
# Install pre-commit framework
pip install pre-commit

# Create .pre-commit-config.yaml
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0   # pin to a specific version
    hooks:
      - id: gitleaks
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: detect-private-key
      - id: check-added-large-files
EOF

pre-commit install
```

#### Summary: non-managed defense layers

| Layer | Enforcement | Can developer bypass? | Catches prompt injection? |
|---|---|---|---|
| `.claude/settings.json` deny rules | Claude Code runtime | Yes (can edit local settings) | Partially — blocks tool use but developer can override |
| CLAUDE.md instructions | Model behavior | N/A (model may not follow if injected) | Weakly — soft defense |
| Pre-commit hooks | Local git hooks | Yes (`--no-verify`) | Yes for secrets, not for logic |
| **CI/CD scanning (SAST, secrets, deps)** | **GitHub/CI platform** | **No (runs server-side)** | **Yes — catches committed issues** |
| **Branch protection + required reviews** | **GitHub/CI platform** | **No (enforced by platform)** | **Yes — human catches what automation misses** |
| **CODEOWNERS** | **GitHub platform** | **No (enforced by platform)** | **Yes — protects governance files** |

The bolded rows are the **hard controls** in a non-managed environment. CI/CD scanning, branch protection, and CODEOWNERS are enforced server-side and cannot be bypassed by a local prompt injection or a compromised developer workstation.

## Standards Alignment

- [Project CodeGuard](https://project-codeguard.org/) (CoSAI/OWASP) for technical code security
- [OWASP Top 10 for LLM Applications (2025)](https://genai.owasp.org/llm-top-10/) for AI application risks
- [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) for autonomous AI risks
- [OWASP GenAI Security Project](https://genai.owasp.org/) for broader governance guidance

## Customization

Before deploying to a client or organization:

- Replace `[COMPANY NAME]` in root CLAUDE.md
- Adjust secrets management tool references to match the client's stack
- Review and customize managed-settings.json deny/ask rules for the org's workflows
- Add or remove approved licenses based on Legal team input
- Update MCP server governance to reflect approved integrations

## License

Apache 2.0
