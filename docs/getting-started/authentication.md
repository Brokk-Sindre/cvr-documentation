# Authentication

The Danish CVR Registry API requires HTTP Basic Authentication for all requests.

## Obtaining Credentials

!!! warning "Credentials Required"
    You must obtain valid credentials from the Danish Business Authority before using the API.

### How to Request Access

Apply via the form on this webpage:
https://datacvr.virk.dk/artikel/system-til-system-adgang-til-cvr-data 

3. **Wait for approval** - Processing may take up to 3 weeks.

## Setting Up Authentication

### Environment Variables (Recommended)

Store your credentials as environment variables for security:

=== "Linux/macOS"
    ```bash
    export CVR_USERNAME="your_username_here"
    export CVR_PASSWORD="your_password_here"
    ```

=== "Windows (PowerShell)"
    ```powershell
    $env:CVR_USERNAME = "your_username_here"
    $env:CVR_PASSWORD = "your_password_here"
    ```

=== "Docker"
    ```bash
    docker run -e CVR_USERNAME="your_username_here" \
               -e CVR_PASSWORD="your_password_here" \
               your-app:latest
    ```

### Configuration Files

For applications, use configuration files with environment variable substitution:

=== "YAML Configuration"
    ```yaml
    cvr:
      username: "${CVR_USERNAME}"
      password: "${CVR_PASSWORD}"
      base_url: "http://distribution.virk.dk/cvr-permanent"
    ```

=== "JSON Configuration"
    ```json
    {
      "cvr": {
        "username": "${CVR_USERNAME}",
        "password": "${CVR_PASSWORD}",
        "base_url": "http://distribution.virk.dk/cvr-permanent"
      }
    }
    ```

### .env Files

Create a `.env` file for local development:

```bash
# .env file (do not commit to version control)
CVR_USERNAME=your_username_here
CVR_PASSWORD=your_password_here
```

Add `.env` to your `.gitignore`:

```bash
echo ".env" >> .gitignore
```

## Testing Authentication

Verify your credentials work correctly:

```bash
curl -u "$CVR_USERNAME:$CVR_PASSWORD" \
  -X POST "http://distribution.virk.dk/cvr-permanent/virksomhed/_search" \
  -H 'Content-Type: application/json' \
  -d '{"size": 1}'
```

### Expected Response

A successful authentication returns company data:

```json
{
  "took": 16,
  "timed_out": false,
  "_shards": {
    "total": 6,
    "successful": 6,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2194982,
    "max_score": 1.0,
    "hits": [
      {
        "_index": "cvr-v-20220630",
        "_type": "_doc",
        "_id": "...",
        "_score": 1.0,
        "_source": {
          "Vrvirksomhed": {
            "cvrNummer": 12345678,
            "navne": [...]
          }
        }
      }
    ]
  }
}
```

## Authentication Errors

### HTTP 401 - Unauthorized

```html
<html>
<head><title>401 Authorization Required</title></head>
<body>
<center><h1>401 Authorization Required</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

**Common Causes**:
- Incorrect username or password
- Expired credentials

**Solutions**:
- Verify credentials with CVR support team
- Check environment variables are set correctly
- Ensure no special characters are causing encoding issues

## Security Best Practices

!!! danger "Never Commit Credentials"
    **Never** store credentials directly in your code or commit them to version control.

### ✅ Do This

- Use environment variables
- Store in secure credential management systems (AWS Secrets Manager, Azure Key Vault, etc.)
- Use configuration files with environment variable substitution
- Implement credential rotation procedures

### ❌ Don't Do This

- Hardcode credentials in source code
- Store credentials in configuration files without environment variables
- Share credentials via email or chat
- Use the same credentials across multiple environments

### Credential Management in Production

=== "AWS Secrets Manager"
    ```python
    import boto3
    import json
    
    def get_cvr_credentials():
        client = boto3.client('secretsmanager', region_name='eu-west-1')
        secret = client.get_secret_value(SecretId='cvr-api-credentials')
        credentials = json.loads(secret['SecretString'])
        return credentials['username'], credentials['password']
    ```

=== "Azure Key Vault"
    ```python
    from azure.keyvault.secrets import SecretClient
    from azure.identity import DefaultAzureCredential
    
    def get_cvr_credentials():
        credential = DefaultAzureCredential()
        client = SecretClient(vault_url="https://your-vault.vault.azure.net/", 
                             credential=credential)
        
        username = client.get_secret("cvr-username").value
        password = client.get_secret("cvr-password").value
        return username, password
    ```

=== "Kubernetes Secrets"
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: cvr-credentials
    type: Opaque
    data:
      username: base64_encoded_username
      password: base64_encoded_password
    ```

### IP Restrictions

!!! info "Network Access"
    The CVR API may have IP-based restrictions. Ensure your production systems can access the API endpoint from their network locations.

If you encounter network connectivity issues:

1. Check if your organization's firewall blocks HTTP (non-HTTPS) traffic
2. Verify the API endpoint is accessible from your deployment environment
3. Consider using a proxy or VPN if required by your network policies

## Next Steps

Once authentication is working:

1. [Quick Start Guide](quick-start.md) - Try basic queries
2. [API Reference](../api-reference/endpoints.md) - Learn about available endpoints  
3. [Query Examples](../queries/basic-queries.md) - See practical query patterns
