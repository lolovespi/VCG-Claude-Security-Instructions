# Security Patterns Reference

Implementation patterns for secure code. These are reference examples loaded on demand, not on every prompt.

## Secrets Management Patterns

### Python (AWS Secrets Manager)

```python
# WRONG - Never generate this
api_key = "sk-abc123xyz789"
db_password = "MyP@ssw0rd!"

# CORRECT - AWS Secrets Manager
import boto3
client = boto3.client('secretsmanager')
secret = client.get_secret_value(SecretId='my-app/api-key')
api_key = secret['SecretString']

# CORRECT - Environment variables for local dev
import os
api_key = os.environ.get('API_KEY')
if not api_key:
    raise ValueError("API_KEY environment variable not set")
```

### iOS (Keychain)

```swift
// WRONG - Never generate this
let apiKey = "sk-abc123xyz789"
UserDefaults.standard.set(token, forKey: "authToken")

// CORRECT - iOS Keychain
import Security

func saveToKeychain(key: String, value: String) {
    let data = value.data(using: .utf8)!
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecValueData as String: data
    ]
    SecItemAdd(query as CFDictionary, nil)
}
```

### Android (EncryptedSharedPreferences)

```kotlin
// WRONG - Never generate this
val apiKey = "sk-abc123xyz789"
sharedPrefs.edit().putString("authToken", token).apply()

// CORRECT - Android EncryptedSharedPreferences
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val encryptedPrefs = EncryptedSharedPreferences.create(
    context, "secure_prefs", masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)
```

## Network Security Patterns

### iOS (ATS and Certificate Pinning)

```swift
// WRONG - Never generate this
// Disabling ATS in Info.plist is WRONG for production
let config = URLSessionConfiguration.default
config.urlCache = nil
```

### Android (SSL Validation)

```kotlin
// WRONG - Never generate this
.hostnameVerifier { _, _ -> true }
.sslSocketFactory(trustAllCerts)
```

## Open Source License Examples

```
# WRONG - Do not suggest without license warning
"Use MongoDB for your database"  # SSPL license

# CORRECT - Flag license concerns
"MongoDB uses SSPL which requires Legal approval.
Alternatives with permissive licenses:
- PostgreSQL (PostgreSQL License, similar to MIT)
- MySQL (GPL, but offers commercial license)
Consider PostgreSQL for this use case."
```

## Infrastructure Anti-Patterns

### Terraform

```hcl
# WRONG - Overly permissive IAM
resource "aws_iam_policy" "bad" {
  policy = jsonencode({
    Statement = [{
      Action   = "*"
      Resource = "*"
      Effect   = "Allow"
    }]
  })
}

# CORRECT - Scoped permissions
resource "aws_iam_policy" "good" {
  policy = jsonencode({
    Statement = [{
      Action   = ["s3:GetObject", "s3:PutObject"]
      Resource = "arn:aws:s3:::my-bucket/*"
      Effect   = "Allow"
    }]
  })
}
```

### Docker

```dockerfile
# WRONG - Running as root with latest tag
FROM node:latest
COPY . /app
RUN npm install

# CORRECT - Pinned version, non-root, multi-stage
FROM node:20.11.1-slim AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
USER node
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1
```

### Kubernetes

```yaml
# WRONG - Privileged container
spec:
  containers:
  - name: app
    securityContext:
      privileged: true
      runAsRoot: true

# CORRECT - Hardened security context
spec:
  containers:
  - name: app
    securityContext:
      runAsNonRoot: true
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
    resources:
      limits:
        cpu: "500m"
        memory: "256Mi"
```
