```yaml
# .github/workflows/falcon-sensor.yml

name: Falcon Sensor ACR Sync

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:
  schedule:
    - cron: '0 * * * *'

jobs:
  sync-sensor:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Verify Environment Variables
        env:
          TEST_CLIENT_ID: ${{ vars.FALCON_CLIENT_ID }}
          TEST_CLIENT_SECRET: ${{ secrets.FALCON_CLIENT_SECRET }}
        run: |
          echo "🔍 Verifying environment variables..."
          if [ -n "$TEST_CLIENT_ID" ]; then
            echo "✅ FALCON_CLIENT_ID is set"
          else
            echo "❌ FALCON_CLIENT_ID is NOT set"
            exit 1
          fi
          if [ -n "$TEST_CLIENT_SECRET" ]; then
            echo "✅ FALCON_CLIENT_SECRET is set"
          else
            echo "❌ FALCON_CLIENT_SECRET is NOT set"
            exit 1
          fi

      - name: Login to Azure Container Registry
        uses: azure/docker-login@v1
        with:
          login-server: <youracr>.azurecr.io
          username: ${{ secrets.ACR_USERNAME }}
          password: ${{ secrets.ACR_PASSWORD }}

      - name: Download and Process Falcon Container Sensor
        env:
          FCSCLIENTID: ${{ vars.FALCON_CLIENT_ID }}
          FSCSECRET: ${{ secrets.FALCON_CLIENT_SECRET }}
        run: |
          echo "⬇️ Downloading Falcon Container Sensor pull script..."
          curl -sSL -o falcon-container-sensor-pull.sh https://github.com/CrowdStrike/falcon-scripts/releases/latest/download/falcon-container-sensor-pull.sh
          chmod 777 ./falcon-container-sensor-pull.sh

          echo "🔄 Copying sensor to ACR..."
          ./falcon-container-sensor-pull.sh \
          --client-id "$FCSCLIENTID" \
          --client-secret "$FSCSECRET" \
          --region us-1 \
          --type falcon-sensor \
          --copy <youracr>.azurecr.io/falcon-sensor

          echo "✅ Sensor processing complete"

      - name: Run FCS IaC Scan
        uses: crowdstrike/fcs-action@v2.0.0
        env:
          INPUT_FALCON_CLIENT_ID: ${{ vars.FALCON_CLIENT_ID }}
          FALCON_CLIENT_SECRET: ${{ secrets.FALCON_CLIENT_SECRET }}
        with:
          falcon_client_id: ${{ vars.FALCON_CLIENT_ID }}
          falcon_region: 'us-1'
          path: './'
```

And here's the updated README:

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

### Check Deployment
```bash
# Login to ACR
az acr login -n <youracr>

# List images
docker images | grep falcon-sensor
```

### View Workflow Status
1. Go to Actions tab
2. Check latest workflow run
3. Review step outputs

## Example Run
<img width="1012" height="544" alt="image" src="https://github.com/user-attachments/assets/fffb034f-335d-4978-b8aa-b603dc2a6ccd" />


## 🛟 Troubleshooting

### Common Issues
1. Authentication Failures
   - Verify secrets are set correctly
   - Check API permissions
   - Confirm ACR credentials

2. ACR Access
   - Verify ACR name is correct
   - Ensure admin access is enabled
   - Check network connectivity

## 📚 References
- [CrowdStrike Docs](https://falcon.crowdstrike.com/documentation/146/falcon-container-security)
- [ACR Documentation](https://learn.microsoft.com/en-us/azure/container-registry/)
- [GitHub Actions](https://docs.github.com/en/actions)

## ⚠️ Important Notes
- Replace all instances of `<youracr>` with your actual ACR name
- Keep secrets and credentials secure
- Test with manual trigger before enabling schedule
```
