# Cloud, Container, and Infrastructure Security Requirements

These rules are mandatory when generating infrastructure-as-code, container configurations, CI/CD pipelines, or cloud resource definitions. They extend the root CLAUDE.md and work alongside Project CodeGuard's cloud security rules.

## Infrastructure as Code (Terraform, CloudFormation, Pulumi)

### IAM and Access Control
- Never generate wildcard IAM policies (Action: "*", Resource: "*").
- Use least-privilege. Scope permissions to specific resources and actions.
- Never attach AdministratorAccess or PowerUser managed policies to service roles.
- Require MFA for console access in IAM policies.
- Use service-linked roles and instance profiles instead of long-lived access keys.

### Storage and Data
- S3 buckets: Block public access by default. Enable versioning and server-side encryption (SSE-S3 or SSE-KMS).
- Database instances: Never set publicly_accessible = true. Enable encryption at rest. Require TLS for connections.
- EBS volumes: Always enable encryption. Never leave default unencrypted.

### Networking
- Security groups: Never allow 0.0.0.0/0 on SSH (22), RDP (3389), or database ports.
- Default VPC: Do not use. Create purpose-built VPCs with proper subnet segmentation.
- Enable VPC flow logs for audit and troubleshooting.

### State and Secrets
- Terraform state files must be stored in encrypted remote backends (S3 + DynamoDB, Azure Blob, GCS).
- Never store secrets in Terraform variables, tfvars files, or state. Use secrets manager references.
- Enable state file locking to prevent concurrent modifications.

## Container Security (Docker)

- Never use `FROM latest`. Pin base images by digest or specific version tag.
- Never run containers as root. Use `USER nonroot` or a specific UID.
- Use multi-stage builds to minimize attack surface.
- Do not copy secrets into images. Use runtime injection via secrets managers or mounted volumes.
- Do not install unnecessary packages, compilers, or debug tools in production images.
- Set `HEALTHCHECK` instructions for production containers.
- Scan images for vulnerabilities before pushing to registry.

```
# WRONG
FROM node:latest
COPY . /app
RUN npm install

# CORRECT
FROM node:20.11.1-slim AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
USER node
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1
```

## Kubernetes Security

### Pod Security
- Never set `privileged: true` on containers.
- Always define a securityContext with `runAsNonRoot: true` and `readOnlyRootFilesystem: true`.
- Drop all capabilities, then add only what is needed: `capabilities: { drop: ["ALL"], add: ["NET_BIND_SERVICE"] }`.
- Set resource limits (CPU, memory) on every container to prevent unbounded consumption.

### Network
- Apply NetworkPolicies with default-deny ingress and egress. Explicitly allow only required traffic.
- Use TLS/mTLS for service-to-service communication.

### Secrets
- Kubernetes Secrets are base64-encoded, not encrypted. Enable encryption at rest for etcd.
- Prefer external secrets operators (AWS Secrets Manager CSI driver, HashiCorp Vault) over native K8s Secrets for sensitive data.
- Never mount ServiceAccount tokens unless required. Set `automountServiceAccountToken: false`.

### RBAC
- Never grant cluster-admin to applications or CI/CD service accounts.
- Use namespace-scoped Roles, not ClusterRoles, when possible.
- Audit RBAC bindings regularly. Remove unused permissions.

## CI/CD Pipeline Security

### GitHub Actions / Pipeline Hardening
- Pin action versions by SHA, not tag: `uses: actions/checkout@abcdef123` not `uses: actions/checkout@v4`.
- Set `permissions` to minimum needed at workflow and job level. Default to `contents: read`.
- Never pass secrets as command-line arguments. Use environment variables or the secrets context.
- Do not allow `pull_request_target` triggers with checkout of PR code (injection risk).

### Artifact Security
- Sign build artifacts and container images (Sigstore/cosign).
- Generate and attach SBOM to every release.
- Store artifacts in authenticated, access-controlled registries.

### Secrets in CI/CD
- Use the platform's secrets manager (GitHub Secrets, GitLab CI Variables, etc.).
- Never echo or print secrets in build logs.
- Rotate CI/CD credentials on a defined schedule.
- Restrict secret access to specific environments and branches.

## Secure Logging and Monitoring

- Log authentication events, authorization failures, and configuration changes.
- Never log secrets, tokens, passwords, PII, or full credit card numbers.
- Use structured logging (JSON) for machine parseability.
- Sanitize log inputs to prevent log injection attacks.
- Send logs to centralized, immutable storage (CloudWatch, Datadog, Splunk).
- Set retention policies aligned with compliance requirements.
