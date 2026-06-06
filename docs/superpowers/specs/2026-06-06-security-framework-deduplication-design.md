# Security Framework Deduplication

## Goal

Reduce within-version content sprawl across the Claude-Security-Instructions framework. Each rule or example has exactly one home; other locations link to it instead of restating it.

Slim/ and root/ remain two **fully independent products** by user decision — accepted duplication between them is not in scope. This spec only addresses duplication *within* each version.

## Source-of-truth assignments

| Content | Source of truth | Locations that become pointers |
|---|---|---|
| Permission deny/ask rules (JSON) | `managed-settings-template.jsonc` (root) and `slim/managed-settings-template.jsonc` (slim) | `README.md`, `slim/README.md`, `docs/security-patterns/agentic-security.md` |
| OWASP LLM Top 10 prescriptive rules + mitigations | `CLAUDE.md` § AI Agent Security (single consolidated table) | `docs/security-patterns/agentic-security.md` (table removed) |
| Application-level audit logging | `CLAUDE.md` § Audit Logging | `infra/CLAUDE.md` § Secure Logging and Monitoring (trims to cloud-specific only) |
| Secrets-management code examples (slim) | `slim/docs/security-patterns/secrets-management.md` (synced full copy of root) | n/a — file becomes self-contained, identical to root |

## File-by-file changes

### `CLAUDE.md` (root, currently 495 lines)

**Change:** In the "AI Agent Security" section, replace the bulleted LLM01–LLM10 sub-sections with a **single consolidated table** that pulls the mitigation and enforcement-level columns from `docs/security-patterns/agentic-security.md`. The table becomes the canonical OWASP LLM Top 10 mapping.

Columns: `OWASP Risk | Rule (prescriptive) | Mitigation | Enforcement Level`

The current bullet structure is 3–5 sub-bullets per LLM number. The table compresses each item into one row: the "Rule" cell becomes a semicolon-joined list of the existing bullet points (lossless — every rule preserved, just reflowed). If any LLM item has a long supporting paragraph (e.g., the "Known injection vectors in coding workflows" callout under LLM01), keep that paragraph immediately above or below the table — don't lose it in the compression.

The "Enforcement levels" explanation paragraph stays in CLAUDE.md (it's a one-time concept introduction). Drop the duplicated key from agentic-security.md.

Keep all other sections of `CLAUDE.md` unchanged.

**Estimated new length:** ~480 lines (roughly neutral — the table is denser than bullets but absorbs columns from agentic-security.md).

### `docs/security-patterns/agentic-security.md` (currently 183 lines)

**Cuts:**
- Remove the OWASP LLM Top 10 mapping table (lines 7–18) — moved to `CLAUDE.md`.
- Remove the "Enforcement level key" block (lines 20–25) — moved to `CLAUDE.md`.
- Remove both inline `permissions` JSON blocks (lines 73–110 and 122–127). Replace each with: *"See `managed-settings-template.jsonc` for the authoritative deny/ask rules and `strictKnownMarketplaces` configuration."*

**Keeps (unique content):**
- Prompt-injection defense-in-depth explanation
- Specific attack scenarios (malicious dependency README, PR modifying CLAUDE.md, encoded curl exfiltration, subtle code injection)
- Defense layers table
- MCP server governance prose (the non-JSON rules)
- Human-in-the-loop requirements
- Compliance API integration notes
- Compliance Framework Mapping table (SOC 2 / ISO 27001 / PCI-DSS / HIPAA / GDPR) — fully unique
- OWASP GenAI references

**Estimated new length:** ~110 lines.

### `README.md` (root, currently 300 lines)

**Change:** In the "Non-Managed / Individual / Small Team" Quick Start section, the project-level `.claude/settings.json` example (lines 70–115) currently inlines ~30 lines of JSON. Shrink to a **6–8 line minimal example** showing the format, then:

> *"For the full enterprise rule set (network egress, secret-file reads, destructive commands), copy from `managed-settings-template.jsonc` and adapt for your project. The template uses the same JSON schema as `.claude/settings.json`."*

Keep the user-level example (lines 117–131) as-is — already small.

**Estimated new length:** ~270 lines.

### `slim/README.md` (currently 93 lines)

**Change:** Same treatment as root `README.md`. Shrink the JSON example (lines 45–69) to a minimal version, link to `slim/managed-settings-template.jsonc` for the full list.

**Estimated new length:** ~75 lines.

### `infra/CLAUDE.md` (currently 153 lines)

**Change:** The "Secure Logging and Monitoring" section (lines 146–153) currently restates application-level audit logging rules already in root `CLAUDE.md` § Audit Logging. Trim to **cloud-specific differences only**:

- Centralized log destinations (CloudWatch, Datadog, Splunk)
- Log injection at ingest (sanitize inputs at the WAF/ALB layer)
- Immutable storage (S3 Object Lock, Azure Immutable Blob)
- Cloud-side retention policy automation

Prefix with: *"For application-level audit logging requirements (which events to log, structured logging, never logging secrets), see root `CLAUDE.md` § Audit Logging. The rules below extend those for cloud and platform layers."*

**Estimated new length:** ~145 lines.

### `slim/docs/security-patterns/secrets-management.md` (currently ~270 lines)

**Change:** Replace contents with an identical copy of `docs/security-patterns/secrets-management.md` (root, 354 lines). This adds iOS/Android/Flutter code examples that were trimmed out.

**Rationale:** Slim is a self-contained product — its `CLAUDE.md` doc link must resolve within `slim/`. Keeping a synced copy is the cost of independence. The user accepted this duplication explicitly.

**Estimated new length:** ~354 lines (matches root).

### `slim/CLAUDE.md` (currently 213 lines)

**Change:** None. Its existing link target (line 43) already resolves within `slim/`.

### Files unchanged

- `mobile/CLAUDE.md` — already minimal and unique, no overlaps to remove.
- `managed-settings-template.jsonc` (root and slim) — these are now the *source of truth*, so unchanged in content.
- `scripts/security-checks.md` — unique CLI commands, no overlap with what's being touched.
- `LICENSE`, `.gitignore` — unchanged.

## Out of scope

- Splitting root `CLAUDE.md` into topic modules (Approach B from brainstorming). Worth doing eventually but a bigger restructure that changes the deployment model.
- Deduplicating between slim/ and root/. User chose to keep them fully independent.
- Re-aligning the framework against newer OWASP releases. Content updates are a separate effort.

## Success criteria

After the changes:

1. Every permission deny/ask rule appears in exactly **one** authoritative file per version (root `managed-settings-template.jsonc`, `slim/managed-settings-template.jsonc`). All other locations either show a short copy-paste-friendly example or a pointer to the template.
2. The OWASP LLM Top 10 prescriptive content appears in exactly **one** file per version (root `CLAUDE.md`).
3. Application-level audit logging rules appear in exactly **one** file per version (root `CLAUDE.md`); `infra/CLAUDE.md` only adds cloud-specific extensions.
4. `slim/docs/security-patterns/secrets-management.md` is byte-identical to `docs/security-patterns/secrets-management.md`.
5. No anchor links or relative paths break. All `[link](path)` references in modified files resolve.
6. Net line count change: roughly **−140 lines** of duplicated content removed (agentic-security.md −70, READMEs −48, infra −8, CLAUDE.md neutral) offset by **+80 lines** added to slim/docs/secrets-management.md to make it a synced copy. Net: ~**−60 lines** across the framework. The point isn't the line count — it's that no rule appears twice.

## Risks and mitigations

- **Risk:** Users who copy `slim/` into projects rely on its self-contained property. Syncing its secrets-management doc to root adds bytes but is harmless. → Accept.
- **Risk:** Future contributors update one `managed-settings-template.jsonc` but not the other (slim vs root drift). → Out of scope here; could be addressed by a CI check in a follow-up.
- **Risk:** OWASP table consolidation changes how the rules read. → Mitigated by preserving every rule as a row; only the *layout* changes.
- **Risk:** Truncating the README JSON example reduces copy-paste convenience for non-MDM users. → Mitigated by leaving a working minimal example; full version is one file open away.
