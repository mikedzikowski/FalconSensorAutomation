# FalconSensorAutomation
Automatically Pull Latest CrowdStrike Falcon Sensor Image and Upload to Azure Container Registrty
```markdown
# 🦅 CrowdStrike Falcon Container Sensor Automation

Simple GitHub Action to automate downloading the CrowdStrike Falcon Container Sensor and pushing it to Azure Container Registry (ACR).

## Setup

### Required Permissions
1. CrowdStrike API Client
   ```yaml
   - Falcon Container Image Download: Read
   - Falcon Container Image: Write
   - Falcon Container Registry: Write
   ```

2. Azure Container Registry Access
   ```yaml
   - Registry Name: <registry>.azurecr.io
   - Admin Access Enabled
   ```

### GitHub Configuration
Add to repository secrets:
```yaml
ACR_USERNAME: Azure Container Registry username
ACR_PASSWORD: Azure Container Registry password
FALCON_CLIENT_SECRET: CrowdStrike API secret
```

Add to repository variables:
```yaml
FALCON_CLIENT_ID: CrowdStrike API client ID
```

## Workflow File

### Location
Save as `.github/workflows/falcon-sensor.yml`

### Structure
```yaml
name: CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:
  schedule:
    - cron: '0 * * * *'  # Runs hourly

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Login to ACR
        uses: azure/docker-login@v1
      - name: Download and Process Falcon Container Sensor
        # Downloads and pushes sensor
      - name: Run FCS IaC Scan
        uses: crowdstrike/fcs-action@v2.0.0
```

### Key Components
1. **Triggers** (`on:`)
   ```yaml
   push: main branch
   pull_request: main branch
   workflow_dispatch: manual run
   schedule: hourly via cron
   ```

2. **ACR Login** (`azure/docker-login@v1`)
   ```yaml
   login-server: <registry>.azurecr.io
   username: ${{ secrets.ACR_USERNAME }}
   password: ${{ secrets.ACR_PASSWORD }}
   ```

3. **Sensor Download**
   ```yaml
   # Script parameters
   --client-id: Falcon API Client ID
   --client-secret: Falcon API Secret
   --region: us-1
   --type: falcon-sensor
   --copy: <registry>.azurecr.io/falcon-sensor
   ```

### Customization
1. Update Registry Name
   ```yaml
   login-server: your-registry.azurecr.io
   ```

2. Change Schedule
   ```yaml
   # Examples
   - cron: '0 * * * *'     # Every hour
   - cron: '0 */2 * * *'   # Every 2 hours
   - cron: '0 0 * * *'     # Daily at midnight
   ```

3. Modify Region
   ```yaml
   --region: us-1  # Change to your Falcon cloud region
   ```

## Usage

### Image Location
```bash
<registry>.azurecr.io/falcon-sensor:latest
```

### Verify Deployment
```bash
# Login to ACR
az acr login -n <registry-name>

# Check image
docker images | grep falcon-sensor
```

### Manual Trigger
1. Go to Actions tab
2. Select workflow
3. Click "Run workflow"

## Troubleshooting

### Common Issues
1. 🔑 Authentication Failures
   ```yaml
   - Check secrets are properly set
   - Verify API client permissions
   - Confirm ACR credentials
   ```

2. 🏷️ Image Push Errors
   ```yaml
   - Verify ACR exists and is accessible
   - Check network connectivity
   - Confirm storage quota
   ```

## Support
- 🔧 Issues: Open in this repo
- 📚 Docs: [CrowdStrike Container Security](https://falcon.crowdstrike.com/documentation/146/falcon-container-security)
- 🐳 ACR: [Azure Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/)
```
