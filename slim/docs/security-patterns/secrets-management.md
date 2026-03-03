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

### Node.js (AWS Secrets Manager)

```javascript
// WRONG - Never generate this
const apiKey = "sk-abc123xyz789";

// CORRECT - AWS Secrets Manager
const { SecretsManagerClient, GetSecretValueCommand } = require("@aws-sdk/client-secrets-manager");
const client = new SecretsManagerClient();
const secret = await client.send(new GetSecretValueCommand({ SecretId: "my-app/api-key" }));
const apiKey = secret.SecretString;
```

### Node.js (Azure Key Vault)

```javascript
// CORRECT - Azure Key Vault
const { SecretClient } = require("@azure/keyvault-secrets");
const { DefaultAzureCredential } = require("@azure/identity");
const client = new SecretClient("https://my-vault.vault.azure.net", new DefaultAzureCredential());
const secret = await client.getSecret("my-api-key");
const apiKey = secret.value;
```

### Python (Google Cloud Secret Manager)

```python
# CORRECT - GCP Secret Manager
from google.cloud import secretmanager

client = secretmanager.SecretManagerServiceClient()
name = "projects/my-project/secrets/my-api-key/versions/latest"
response = client.access_secret_version(request={"name": name})
api_key = response.payload.data.decode("UTF-8")
```

### Go (Environment Variables / Secrets Manager)

```go
// WRONG - Never generate this
apiKey := "sk-abc123xyz789"

// CORRECT - Environment variable
apiKey := os.Getenv("API_KEY")
if apiKey == "" {
    log.Fatal("API_KEY environment variable not set")
}

// CORRECT - AWS Secrets Manager
cfg, err := config.LoadDefaultConfig(context.TODO())
if err != nil {
    log.Fatalf("unable to load AWS config: %v", err)
}
client := secretsmanager.NewFromConfig(cfg)
result, err := client.GetSecretValue(context.TODO(), &secretsmanager.GetSecretValueInput{
    SecretId: aws.String("my-app/api-key"),
})
if err != nil {
    log.Fatalf("unable to retrieve secret: %v", err)
}
apiKey = *result.SecretString
```

### Java (Azure Key Vault)

```java
// WRONG - Never generate this
String apiKey = "sk-abc123xyz789";

// CORRECT - Azure Key Vault
SecretClient client = new SecretClientBuilder()
    .vaultUrl("https://my-vault.vault.azure.net")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
String apiKey = client.getSecret("my-api-key").getValue();
```

## Database Security Patterns

### Parameterized Queries

```python
# WRONG - SQL injection vulnerability
query = f"SELECT * FROM users WHERE email = '{user_input}'"
cursor.execute(query)

# CORRECT - Parameterized query
cursor.execute("SELECT id, name, email FROM users WHERE email = %s", (user_input,))
```

```javascript
// WRONG - SQL injection vulnerability
const query = `SELECT * FROM users WHERE id = ${req.params.id}`;
db.query(query);

// CORRECT - Parameterized query
db.query("SELECT id, name, email FROM users WHERE id = $1", [req.params.id]);
```

```java
// WRONG - SQL injection vulnerability
String query = "SELECT * FROM users WHERE id = " + userId;
stmt.executeQuery(query);

// CORRECT - Prepared statement
PreparedStatement stmt = conn.prepareStatement("SELECT id, name, email FROM users WHERE id = ?");
stmt.setInt(1, userId);
ResultSet rs = stmt.executeQuery();
```

### ORM Safety

```python
# WRONG - Raw SQL in ORM
User.objects.raw(f"SELECT * FROM users WHERE name = '{name}'")

# CORRECT - ORM query methods
User.objects.filter(name=name).values("id", "name", "email")
```

### Database Connection Security

```python
# WRONG - Hardcoded connection string
conn = psycopg2.connect("postgresql://admin:password123@db.internal:5432/mydb")

# CORRECT - Secrets manager + TLS
import json
import boto3
secret = boto3.client('secretsmanager').get_secret_value(SecretId='db-credentials')
creds = json.loads(secret['SecretString'])
conn = psycopg2.connect(
    host=creds['host'], port=creds['port'],
    user=creds['username'], password=creds['password'],
    dbname=creds['dbname'], sslmode='verify-full',
    sslrootcert='/path/to/rds-ca-bundle.pem'
)
```
