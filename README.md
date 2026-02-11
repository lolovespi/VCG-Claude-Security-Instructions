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

## Quick Start

1. Clone this repo
2. Customize `[COMPANY NAME]` placeholder in `CLAUDE.md`
3. Install Project CodeGuard: `/plugin marketplace add cosai-oasis/project-codeguard`
4. Deploy `managed-settings-template.jsonc` via MDM (remove comment keys first)
5. Copy the CLAUDE.md structure into your project repositories

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

MIT
