# PAC CLI Reference - Power Apps Code Apps

Complete reference for Power Platform CLI commands used in Code Apps development.

## Installation

```bash
# Install globally
npm install -g @microsoft/powerplatform-cli

# Or use specific version
npm install -g @microsoft/powerplatform-cli@2.6.0

# Verify installation
pac --version
```

## Authentication Commands

### Create Authentication Profile

```bash
# Interactive login (opens browser)
pac auth create

# With environment ID
pac auth create --environment <ENVIRONMENT_ID>

# Service Principal (for CI/CD)
pac auth create \
  --applicationId <CLIENT_ID> \
  --clientSecret <CLIENT_SECRET> \
  --tenant <TENANT_ID> \
  --environment <ENVIRONMENT_URL>

# Certificate-based auth
pac auth create \
  --applicationId <CLIENT_ID> \
  --certificateFile <PATH_TO_CERT> \
  --certificatePassword <PASSWORD> \
  --tenant <TENANT_ID> \
  --environment <ENVIRONMENT_URL>
```

### Manage Authentication Profiles

```bash
# List all profiles
pac auth list

# Show current profile
pac auth who

# Select profile
pac auth select --index <INDEX>

# Clear all profiles
pac auth clear

# Remove specific profile
pac auth remove --index <INDEX>
```

## Environment Commands

```bash
# List available environments
pac env list

# Select environment
pac env select --environment <ENVIRONMENT_URL>

# Show current environment
pac env who
```

## Code App Commands

### Initialize Code App

```bash
# Initialize in current directory
pac code init --displayName "My Code App"

# With specific name
pac code init --displayName "My App" --uniqueName "my_code_app"
```

### Data Source Management

```bash
# List available connections
pac connection list

# Add data source (connector)
pac code add-data-source -a <API_NAME> -c <CONNECTION_ID>

# Examples:
# SQL Server
pac code add-data-source -a sql -c <CONNECTION_ID>

# Dataverse
pac code add-data-source -a dataverse -c <CONNECTION_ID>

# Office 365 Users
pac code add-data-source -a office365users -c <CONNECTION_ID>

# SharePoint
pac code add-data-source -a sharepointonline -c <CONNECTION_ID>

# List added data sources
pac code list-data-sources

# Remove data source
pac code remove-data-source -a <API_NAME>
```

### Build and Deploy

```bash
# Build project (run after npm run build)
pac code build

# Deploy to Power Platform
pac code push

# Deploy to specific environment
pac code push --environment <ENVIRONMENT_URL>

# Deploy with specific solution
pac code push --solution <SOLUTION_NAME>

# Get app info
pac code show

# Download app
pac code download --path <OUTPUT_PATH>
```

### Development

```bash
# Start local development (runs alongside npm run dev)
pac code run

# Watch for changes
pac code watch

# Open in browser
pac code browse
```

## Solution Management (ALM)

```bash
# Create solution
pac solution init --publisher-name "My Publisher" --publisher-prefix "myprefix"

# Add code app to solution
pac solution add-reference --path <CODE_APP_PATH>

# Export solution
pac solution export --path <OUTPUT_PATH> --name <SOLUTION_NAME>

# Import solution
pac solution import --path <SOLUTION_ZIP>

# Publish customizations
pac solution publish
```

## Connector Commands

```bash
# List available connectors
pac connector list

# Create custom connector
pac connector create --path <CONNECTOR_FOLDER>

# Update custom connector
pac connector update --path <CONNECTOR_FOLDER>

# Download connector
pac connector download --name <CONNECTOR_NAME> --path <OUTPUT_PATH>
```

## Plugin/Extension Commands

```bash
# List installed plugins
pac plugin list

# Install plugin
pac plugin install --path <PLUGIN_PATH>

# Update plugin
pac plugin update --name <PLUGIN_NAME>
```

## Utility Commands

```bash
# Show version
pac --version

# Show help
pac --help

# Show help for specific command
pac code --help
pac auth --help
pac solution --help

# Enable debug logging
pac --debug code push

# Set output format
pac auth list --output json
pac env list --output table
```

## Common Workflows

### New Project Setup

```bash
# 1. Create project from template
npx degit microsoft/PowerAppsCodeApps/templates/starter my-app
cd my-app
npm install

# 2. Authenticate
pac auth create --environment <ENV_ID>

# 3. Initialize code app
pac code init --displayName "My App"

# 4. Add data sources
pac connection list
pac code add-data-source -a dataverse -c <CONN_ID>
pac code add-data-source -a sql -c <CONN_ID>

# 5. Develop locally
npm run dev
# In another terminal:
pac code run

# 6. Deploy
npm run build
pac code push
```

### CI/CD Pipeline Commands

```bash
# In GitHub Actions / Azure DevOps
pac auth create \
  --applicationId $APP_ID \
  --clientSecret $CLIENT_SECRET \
  --tenant $TENANT_ID \
  --environment $ENV_URL

npm run build
pac code push
```

### Multi-Environment Deployment

```bash
# Development
pac auth create --environment <DEV_ENV_URL>
pac code push

# Test
pac auth create --environment <TEST_ENV_URL>
pac code push

# Production
pac auth create --environment <PROD_ENV_URL>
pac code push
```

### Troubleshooting Commands

```bash
# Check authentication status
pac auth list
pac auth who

# Verify environment access
pac env list
pac org who

# Debug deployment
pac --debug code push

# Check solution status
pac solution list
```

## Supported Connectors (API Names)

| Connector | API Name | Description |
|-----------|----------|-------------|
| SQL Server | `sql` | Azure SQL, SQL Server |
| Dataverse | `dataverse` | Microsoft Dataverse |
| Office 365 Users | `office365users` | User profiles, photos |
| Office 365 Groups | `office365groups` | Groups, teams |
| SharePoint | `sharepointonline` | Lists, libraries |
| OneDrive | `onedriveforbusiness` | Files, folders |
| Microsoft Teams | `teams` | Teams, channels |
| Azure Data Explorer | `azuredataexplorer` | Kusto queries |
| Power BI | `powerbi` | Reports, datasets |

## Error Codes

| Code | Meaning | Solution |
|------|---------|----------|
| `ENOTFOUND` | Environment URL not found | Check environment URL |
| `EACCES` | Permission denied | Check PAC CLI permissions |
| `EAUTH` | Authentication failed | Re-authenticate with `pac auth create` |
| `ECONNREFUSED` | Cannot connect to Power Platform | Check network/firewall |
| `EINVALID` | Invalid parameters | Check command syntax |

## Best Practices

1. **Use service principals for CI/CD** - Not interactive accounts
2. **Separate auth profiles per environment** - Dev, Test, Prod
3. **Pin PAC CLI version** - Avoid breaking changes
4. **Use `--debug` for troubleshooting** - Detailed logs
5. **Store secrets securely** - GitHub Secrets, Azure Key Vault
6. **Test auth before deployment** - `pac auth list` in pipeline

## References

- [PAC CLI Documentation](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference)
- [Code Apps CLI Reference](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/code)
- [Authentication Guide](https://learn.microsoft.com/en-us/power-platform/developer/cli/authentication)