# CI/CD Pipeline - Power Apps Code Apps

Complete guide for setting up continuous integration and deployment for Power Apps Code Apps using GitHub Actions.

## Overview

The CI/CD pipeline validates code quality, builds the project, and optionally deploys to Power Platform environments.

## Workflow File (`.github/workflows/validate.yml`)

```yaml
name: Validate Power Apps Code App

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  validate:
    name: Validate Code App
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run TypeScript type checking
        run: npm run typecheck

      - name: Run ESLint
        run: npm run lint

      - name: Check Prettier formatting
        run: npm run format:check

      - name: Build project
        run: npm run build

      - name: Verify Power Apps Code Apps configuration
        run: |
          echo "🔍 Verifying Power Apps Code Apps specific configuration..."
          
          # Check tsconfig.json
          if ! grep -q '"verbatimModuleSyntax": false' tsconfig.json; then
            echo "❌ tsconfig.json missing verbatimModuleSyntax: false"
            exit 1
          fi
          echo "✅ tsconfig.json has verbatimModuleSyntax: false"
          
          # Check vite.config.ts
          if ! grep -q 'base: "./"' vite.config.ts; then
            echo "❌ vite.config.ts missing base: \"./\""
            exit 1
          fi
          echo "✅ vite.config.ts has base: \"./\""
          
          # Check for PowerProvider
          if [ ! -f "src/PowerProvider.tsx" ]; then
            echo "⚠️  PowerProvider.tsx not found"
          else
            echo "✅ PowerProvider.tsx exists"
          fi
          
          # Check for generated folder
          if [ ! -d "src/generated" ]; then
            echo "ℹ️  src/generated folder not found (run 'pac code add-data-source' to generate)"
          else
            echo "✅ src/generated folder exists"
          fi

      - name: Install PAC CLI
        run: npm install -g @microsoft/powerplatform-cli

      - name: Verify PAC CLI
        run: pac --version

  # Optional: Deploy to dev environment on main branch
  deploy-dev:
    name: Deploy to Dev Environment
    needs: validate
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    timeout-minutes: 15
    environment: development
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build

      - name: Install PAC CLI
        run: npm install -g @microsoft/powerplatform-cli

      - name: Authenticate with Power Platform
        env:
          PPAC_APP_ID: ${{ secrets.PPAC_APP_ID }}
          PPAC_CLIENT_SECRET: ${{ secrets.PPAC_CLIENT_SECRET }}
          PPAC_TENANT_ID: ${{ secrets.PPAC_TENANT_ID }}
          PPAC_ENVIRONMENT_URL: ${{ secrets.PPAC_ENVIRONMENT_URL }}
        run: |
          pac auth create --applicationId $PPAC_APP_ID --clientSecret $PPAC_CLIENT_SECRET --tenant $PPAC_TENANT_ID --environment $PPAC_ENVIRONMENT_URL

      - name: Deploy Code App
        run: pac code push
```

## Required GitHub Secrets

Configure these in **Repository Settings → Secrets and variables → Actions**:

| Secret | Description | Required |
|--------|-------------|----------|
| `PPAC_APP_ID` | Azure AD App Registration Client ID | For deployment |
| `PPAC_CLIENT_SECRET` | Client Secret for the App Registration | For deployment |
| `PPAC_TENANT_ID` | Azure AD Tenant ID | For deployment |
| `PPAC_ENVIRONMENT_URL` | Power Platform Environment URL (e.g., `https://org.crm.dynamics.com`) | For deployment |

## Azure AD App Registration Setup

### 1. Create App Registration

1. Go to [Azure Portal](https://portal.azure.com) → **Microsoft Entra ID** → **App registrations**
2. Click **New registration**
3. Name: `Power Apps Code Apps CI/CD`
4. Supported account types: **Accounts in this organizational directory only**
5. Click **Register**

### 2. Configure Permissions

1. Go to **API permissions** → **Add a permission**
2. Select **Dynamics CRM** → **Delegated permissions**
3. Check **user_impersonation**
4. Click **Add permissions**
5. Click **Grant admin consent**

### 3. Create Client Secret

1. Go to **Certificates & secrets** → **New client secret**
2. Description: `GitHub Actions CI/CD`
3. Expires: **24 months** (or your policy)
4. Copy the **Value** immediately (shown only once)

### 4. Get Tenant ID

1. Go to **Overview** → Copy **Directory (tenant) ID**

### 5. Get Environment URL

1. Go to [Power Platform Admin Center](https://admin.powerplatform.microsoft.com)
2. Select your environment
3. Copy the **Environment URL** (format: `https://org.crm.dynamics.com`)

## Multi-Environment Deployment

### Separate Workflows per Environment

```yaml
# .github/workflows/deploy-dev.yml
name: Deploy to Development

on:
  push:
    branches: [develop]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: development
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci && npm run build
      - run: npm install -g @microsoft/powerplatform-cli
      - name: Authenticate
        env:
          PPAC_APP_ID: ${{ secrets.PPAC_APP_ID }}
          PPAC_CLIENT_SECRET: ${{ secrets.PPAC_CLIENT_SECRET_DEV }}
          PPAC_TENANT_ID: ${{ secrets.PPAC_TENANT_ID }}
          PPAC_ENVIRONMENT_URL: ${{ secrets.PPAC_ENV_URL_DEV }}
        run: |
          pac auth create --applicationId $PPAC_APP_ID --clientSecret $PPAC_CLIENT_SECRET_DEV --tenant $PPAC_TENANT_ID --environment $PPAC_ENV_URL_DEV
      - run: pac code push
```

```yaml
# .github/workflows/deploy-prod.yml
name: Deploy to Production

on:
  workflow_dispatch:  # Manual trigger only
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci && npm run build
      - run: npm install -g @microsoft/powerplatform-cli
      - name: Authenticate
        env:
          PPAC_APP_ID: ${{ secrets.PPAC_APP_ID }}
          PPAC_CLIENT_SECRET: ${{ secrets.PPAC_CLIENT_SECRET_PROD }}
          PPAC_TENANT_ID: ${{ secrets.PPAC_TENANT_ID }}
          PPAC_ENVIRONMENT_URL: ${{ secrets.PPAC_ENV_URL_PROD }}
        run: |
          pac auth create --applicationId $PPAC_APP_ID --clientSecret $PPAC_CLIENT_SECRET_PROD --tenant $PPAC_TENANT_ID --environment $PPAC_ENV_URL_PROD
      - run: pac code push
```

### Environment-Specific Secrets

| Environment | Secrets |
|-------------|---------|
| Development | `PPAC_CLIENT_SECRET_DEV`, `PPAC_ENV_URL_DEV` |
| Test | `PPAC_CLIENT_SECRET_TEST`, `PPAC_ENV_URL_TEST` |
| Production | `PPAC_CLIENT_SECRET_PROD`, `PPAC_ENV_URL_PROD` |

## Branch Protection Rules

Configure in **Repository Settings → Branches**:

### Main Branch
- Require pull request reviews: **1**
- Require status checks to pass: **Validate Code App**
- Require branches to be up to date: **Yes**
- Include administrators: **Yes**

### Develop Branch
- Require pull request reviews: **1**
- Require status checks to pass: **Validate Code App**

## Deployment Strategies

### 1. Automatic (Main Branch)
- Push to `main` → Auto-deploy to dev
- Manual promotion to test/prod

### 2. Manual (Release Tags)
- Create release tag → Deploy to prod
- Full control over production deployments

### 3. Environment Promotion
```bash
# Dev → Test
pac code push --environment <TEST_ENV_URL>

# Test → Prod
pac code push --environment <PROD_ENV_URL>
```

## Monitoring & Notifications

### Add Teams/Slack Notifications

```yaml
- name: Notify on failure
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: failure
    channel: '#deployments'
    text: "Code App deployment failed for ${{ github.repository }}"
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### Add Deployment Status Badge

```markdown
# In README.md
![Validate](https://github.com/owner/repo/actions/workflows/validate.yml/badge.svg)
![Deploy Dev](https://github.com/owner/repo/actions/workflows/deploy-dev.yml/badge.svg)
```

## Troubleshooting

### PAC CLI Authentication Fails

```bash
# Check secrets are set correctly
echo $PPAC_APP_ID
echo $PPAC_TENANT_ID

# Verify app registration has correct permissions
# Go to Azure Portal → App Registration → API permissions
```

### Build Fails in CI but Works Locally

```yaml
# Ensure Node version matches
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'  # Match your local version
    cache: 'npm'
```

### Deployment Timeout

```yaml
# Increase timeout
deploy-dev:
  timeout-minutes: 20  # Default is 15
```

### PAC CLI Version Issues

```yaml
# Pin specific version
- name: Install PAC CLI
  run: npm install -g @microsoft/powerplatform-cli@2.6.0
```

## Azure DevOps Pipeline (Alternative to GitHub Actions)

Azure DevOps can be used instead of GitHub Actions for CI/CD with Power Apps Code Apps. The equivalent pipeline is defined in an `azure-pipelines.yml` file at the repository root.

### Differences vs GitHub Actions

| Aspect | GitHub Actions | Azure DevOps |
|--------|---------------|--------------|
| **File** | `.github/workflows/validate.yml` | `azure-pipelines.yml` (root of repo) |
| **Secrets** | GitHub Secrets (Settings → Secrets) | Library → Variable Groups + Service Connections |
| **Agent** | `runs-on: ubuntu-latest` (hosted) | `pool: vmImage: ubuntu-latest` (hosted) |
| **Extensions** | Marketplace Actions | Marketplace Tasks (e.g., PowerPlatformToolInstaller) |
| **PP Auth** | Manual PAC CLI with secrets | `PowerPlatformSetConnectionVariables@2` native task |
| **Variable Groups** | N/A | `variables: - group: 'name'` |

### Azure Pipeline Configuration (azure-pipelines.yml)

```yaml
trigger:
  branches:
    include:
      - main
      - develop

pr:
  branches:
    include:
      - main
      - develop

pool:
  vmImage: ubuntu-latest

variables:
  - group: 'PowerPlatform-CodeApps'

stages:
  - stage: Validate
    displayName: 'Validate Code App'
    jobs:
      - job: Validate
        displayName: 'Validate Code App'
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: '20.x'
            displayName: 'Install Node.js 20'

          - script: npm ci
            displayName: 'Install dependencies'

          - script: npm run typecheck
            displayName: 'Run TypeScript type checking'

          - script: npm run lint
            displayName: 'Run ESLint'

          - script: npm run format:check
            displayName: 'Check Prettier formatting'

          - script: npm run build
            displayName: 'Build project'

          - script: |
              echo "Verifying Power Apps Code Apps configuration..."
              if ! grep -q '"verbatimModuleSyntax": false' tsconfig.json; then
                echo "❌ tsconfig.json missing verbatimModuleSyntax: false"
                exit 1
              fi
              echo "✅ tsconfig.json OK"
              if ! grep -q 'base: "./"' vite.config.ts; then
                echo "❌ vite.config.ts missing base: \"./\""
                exit 1
              fi
              echo "✅ vite.config.ts OK"
            displayName: 'Verify Code Apps configuration'

          - task: PowerPlatformToolInstaller@2
            displayName: 'Install PAC CLI'
            inputs:
              DefaultVersion: true

          - script: pac --version
            displayName: 'Verify PAC CLI'

  - stage: DeployDev
    displayName: 'Deploy to Development'
    dependsOn: Validate
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: Deploy
        displayName: 'Deploy Code App'
        environment: 'development'
        strategy:
          runOnce:
            deploy:
              steps:
                - checkout: self
                - script: npm ci && npm run build
                  displayName: 'Build project'
                - task: PowerPlatformToolInstaller@2
                  displayName: 'Install PAC CLI'
                - task: PowerPlatformSetConnectionVariables@2
                  displayName: 'Authenticate with Power Platform'
                  inputs:
                    authenticationType: 'PowerPlatformSPN'
                    PowerPlatformSPN: '$(PowerPlatformServiceConnection)'
                - script: pac code push
                  displayName: 'Deploy Code App'
```

### Setup in Azure DevOps

1. **Create Variable Group**: Pipelines → Library → + Variable Group → `PowerPlatform-CodeApps`
2. **Add variables**: `PPAC_APP_ID`, `PPAC_CLIENT_SECRET`, `PPAC_TENANT_ID`, `PPAC_ENVIRONMENT_URL`
3. **Create Service Connection**: Project Settings → Service Connections → Power Platform (authentication with SPN)
4. **Add `azure-pipelines.yml`** to the root of your repository
5. The pipeline runs automatically on every push and PR

## Best Practices

1. **Use separate secrets per environment** - Never share prod secrets with dev
2. **Require manual approval for production** - Use `workflow_dispatch` or environment protection rules
3. **Pin dependency versions** - Avoid unexpected breaking changes
4. **Cache node_modules** - Speeds up CI runs
5. **Run validation on PRs** - Catch issues before merge
6. **Monitor deployment health** - Set up alerts for failed deployments

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Azure Pipelines Documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/)
- [PAC CLI Reference](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference)
- [Power Platform Environments](https://learn.microsoft.com/en-us/power-platform/admin/environments-overview)
- [Azure AD App Registration](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app)
- [PowerPlatformToolInstaller Task](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/power-platform-tool-installer-v2)

## Sources

| Source | Description | Link |
|--------|-------------|------|
| **Microsoft Power Platform CLI** | PAC CLI reference for CI/CD authentication and deployment | [learn.microsoft.com](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/code) |
| **GitHub Actions Documentation** | Workflow syntax, triggers, and environments | [docs.github.com](https://docs.github.com/en/actions) |
| **Azure Pipelines Documentation** | Azure DevOps pipeline syntax and tasks | [learn.microsoft.com](https://learn.microsoft.com/en-us/azure/devops/pipelines/) |
| **Power Platform Admin Center** | Environment management and service principal setup | [admin.powerplatform.microsoft.com](https://admin.powerplatform.microsoft.com/) |
| **Azure AD App Registration** | Service principal creation and permissions | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app) |
| **PowerPlatformToolInstaller** | Azure DevOps task for PAC CLI installation | [learn.microsoft.com](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/power-platform-tool-installer-v2) |
| **Power Apps Code Apps Repository** | Official deployment patterns and pac code push examples | [github.com/microsoft/PowerAppsCodeApps](https://github.com/microsoft/PowerAppsCodeApps) |

### Pipeline Patterns Attribution

- **Validation workflow** based on [GitHub Actions workflow syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- **PAC CLI authentication** based on [Microsoft service principal authentication](https://learn.microsoft.com/en-us/power-platform/developer/cli/authentication)
- **Multi-environment deployment pattern** based on [Power Platform ALM](https://learn.microsoft.com/en-us/power-platform/alm/) best practices