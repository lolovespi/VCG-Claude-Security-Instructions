# AI Agent Security Patterns

Reference guide for securing AI coding agent deployments. Maps OWASP Top 10 for LLM Applications (2025) and OWASP Top 10 for Agentic Applications to Claude Code enterprise use.

## OWASP LLM Top 10 (2025) Mapped to Claude Code

| OWASP Risk | Claude Code Relevance | Mitigation |
|---|---|---|
| LLM01: Prompt Injection | Malicious content in repos, PRs, or dependencies could manipulate Claude Code behavior | Use managed-settings.json to restrict file reads. Review CLAUDE.md changes in PRs. Do not auto-approve code from untrusted sources. |
| LLM02: Sensitive Information Disclosure | Claude Code may output secrets, PII, or internal architecture details in generated code | Enforce secrets management rules. Scan generated code for secrets pre-commit. Use managed-settings.json to deny reading .env and credential files. |
| LLM03: Supply Chain | Claude Code may suggest vulnerable, abandoned, or typosquatted packages | Require license and vulnerability verification for all dependency suggestions. Pin versions. Use Dependabot and SBOM generation. |
| LLM04: Data and Model Poisoning | Malicious training data is an Anthropic-level concern, not org-level | Monitor Anthropic's security advisories. Use the Compliance API to audit outputs. |
| LLM05: Improper Output Handling | Generated code may not sanitize its own outputs properly | CLAUDE.md rules require input validation and output escaping. CodeGuard rules reinforce this at the code pattern level. |
| LLM06: Excessive Agency | Claude Code can execute commands, modify files, and access infrastructure | Use managed-settings.json to deny destructive commands. Disable bypassPermissions mode. Restrict tool access with allow/deny rules. |
| LLM07: System Prompt Leakage | CLAUDE.md files contain proprietary security policies | Treat CLAUDE.md as internal config. Do not commit sensitive policy details to public repos. Use managed-settings.json for the most sensitive rules. |
| LLM08: Vector and Embedding Weaknesses | Not directly applicable to Claude Code | N/A for most deployments. |
| LLM09: Misinformation | Claude Code may generate plausible but incorrect security implementations | Require code review for all AI-generated code. Use CodeGuard rules as a validation layer. Do not auto-merge AI-generated PRs. |
| LLM10: Unbounded Consumption | Uncontrolled Claude Code usage drives costs and token burn | Use Anthropic's per-user spend caps. Monitor usage analytics. Set token limits in managed-settings.json environment variables. |

## Agentic Security Controls for Claude Code

### Tool Permission Governance

Managed-settings.json should enforce the minimum permissions needed:

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(ssh:*)",
      "Bash(chmod 777:*)",
      "Bash(git push --force:*)",
      "Read(**/.env)",
      "Read(**/*secret*)",
      "Read(**/*credential*)"
    ],
    "disableBypassPermissionsMode": "disable"
  }
}
```

Customize deny rules based on your organization's risk tolerance and workflows.

### MCP Server Governance

Before connecting Claude Code to any MCP server:

1. **Verify the server source.** Only connect to company-approved MCP servers from vetted publishers.
2. **Audit tool permissions.** Review what tools the MCP server exposes before granting access.
3. **Use the Cisco MCP Scanner** (https://github.com/ArcadeAI/mcp-scanner) to analyze MCP connections for security risks.
4. **Never pass credentials through MCP tool parameters.** Use the platform's auth mechanism.
5. **Restrict MCP servers in managed-settings.json.** Use strictKnownMarketplaces to limit plugin sources.

```json
{
  "strictKnownMarketplaces": ["https://approved-marketplace.company.com"]
}
```

### Human-in-the-Loop Requirements

- AI-generated PRs must be reviewed by a human before merge.
- Destructive operations (database migrations, infrastructure changes, production deployments) require explicit human approval.
- Claude Code should not be configured to auto-approve its own suggestions in CI/CD pipelines.

### Compliance API Integration

Anthropic's Compliance API (Enterprise plan) provides:
- Real-time access to Claude Code usage data and content
- Filtering by user and time range
- Selective deletion for data retention compliance

Integrate the Compliance API with your existing:
- SIEM (Splunk, Sentinel, Datadog)
- DLP tooling
- Audit log aggregation
- Incident response workflows

## OWASP GenAI Security Resources

For deeper guidance on these topics:
- OWASP Top 10 for LLM Applications (2025): https://genai.owasp.org/llm-top-10/
- OWASP Top 10 for Agentic Applications: https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/
- OWASP Practical Guide to Securing Agentic Applications: https://genai.owasp.org/initiatives/agentic-security-initiative/
- OWASP MCP Security Guide: https://genai.owasp.org/initiatives/agentic-security-initiative/
- OWASP GenAI Governance Checklist: https://genai.owasp.org/resource/llm-applications-cybersecurity-and-governance-checklist-english/
- Project CodeGuard: https://project-codeguard.org/
