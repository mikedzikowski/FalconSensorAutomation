```markdown
# 🦅 CrowdStrike Falcon Container Sensor Automation

Automates downloading the CrowdStrike Falcon Container Sensor and pushing it to Azure Container Registry (ACR).

## 🚀 Quick Start

1. Fork this repository
2. Replace `<youracr>` in workflow file with your ACR name
3. Configure secrets and variables
4. Enable and run the workflow

## ⚙️ Setup

### Prerequisites
1. Azure Container Registry
2. CrowdStrike Falcon API Credentials
3. GitHub Repository
4. Azure CLI installed (for generating credentials)

### Azure Configuration

1. Create Azure Service Principal and get credentials:
```bash
# Login to Azure
az login

# Create Service Principal with ACR push role
az ad sp create-for-rbac \
  --name "GitHub-ACR-Access" \
  --role AcrPush \
  --scopes /subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.ContainerRegistry/registries/<youracr> \
  --sdk-auth

# This will output JSON similar to:
{
  "clientId": "xxx",
  "clientSecret": "xxx",
  "subscriptionId": "xxx",
  "tenantId": "xxx",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

2. Get ACR credentials:
```bash
# Get ACR admin credentials
az acr credential show -n <youracr>
```

### Required Permissions
1. CrowdStrike API Client
   ```yaml
   - Falcon Container Image Download: Read
   - Falcon Container Image: Write
   - Falcon Container Registry: Write
   ```

2. Azure Container Registry Access
   ```yaml
   - Registry Name: <youracr>.azurecr.io
   - Admin Access Enabled
   - Service Principal: AcrPush role
   ```

### GitHub Configuration
Add to repository secrets:
```yaml
ACR_USERNAME: Azure Container Registry username
ACR_PASSWORD: Azure Container Registry password
FALCON_CLIENT_SECRET: CrowdStrike API secret
AZURE_CREDENTIALS: Entire JSON output from az ad sp create-for-rbac
```

Add to repository variables:
```yaml
FALCON_CLIENT_ID: CrowdStrike API client ID
```

## 📋 Workflow Configuration

### File Location
`.github/workflows/falcon-sensor.yml`

### Required Updates
1. Replace ACR placeholder:
   ```yaml
   # From
   login-server: <youracr>.azurecr.io
   
   # To
   login-server: your-actual-acr.azurecr.io
   ```

2. Update region if needed:
   ```yaml
   --region us-1  # Change to your Falcon cloud region
   ```

### Schedule
Runs hourly by default:
```yaml
schedule:
  - cron: '0 * * * *'
```

## 🔍 Verification

### Check Azure Setup
```bash
# Verify Service Principal
az ad sp list --display-name "GitHub-ACR-Access"

# Test ACR access
az acr login -n <youracr>
```

### Check Deployment
```bash
# Login to ACR
az acr login -n <youracr>

# List images
docker images | grep falcon-sensor

# Check ACR contents
az acr repository list -n <youracr>
```

### View Workflow Status
1. Go to Actions tab
2. Check latest workflow run
3. Review step outputs

## 🛟 Troubleshooting

### Common Issues
1. Authentication Failures
   - Verify secrets are set correctly
   - Check API permissions
   - Confirm ACR credentials
   - Validate Azure JSON format

2. ACR Access
   - Verify ACR name is correct
   - Ensure admin access is enabled
   - Check network connectivity
   - Confirm Service Principal permissions

3. Azure JSON Issues
   ```bash
   # Regenerate credentials if needed
   az ad sp create-for-rbac --name "GitHub-ACR-Access" --role AcrPush --scopes /subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.ContainerRegistry/registries/<youracr> --sdk-auth
   ```

## 📚 References
- [CrowdStrike Docs](https://falcon.crowdstrike.com/documentation/146/falcon-container-security)
- [ACR Documentation](https://learn.microsoft.com/en-us/azure/container-registry/)
- [GitHub Actions](https://docs.github.com/en/actions)
- [Azure Service Principal](https://learn.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals)

## ⚠️ Important Notes
- Replace all instances of `<youracr>` with your actual ACR name
- Keep secrets and credentials secure
- Store Azure JSON output securely
- Test with manual trigger before enabling schedule
- Regularly rotate Azure credentials
```
