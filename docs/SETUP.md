# Setup Guide - Power Apps Code Apps Best Practices

Complete installation and configuration guide for integrating these best practices into your Power Apps Code Apps projects.

## Prerequisites

Before starting, ensure you have:

- **Node.js LTS** (v18.x or v20.x) - [Download](https://nodejs.org/)
- **Power Platform CLI (PAC CLI)** - [Installation Guide](https://learn.microsoft.com/en-us/power-platform/developer/cli/introduction)
- **Git** - [Download](https://git-scm.com/)
- **VS Code** (recommended) - [Download](https://code.visualstudio.com/)
- **Power Apps Premium License** - Required for end users

## Installation Options

### Option 1: New Project (Recommended)

Use Microsoft's official starter template:

```bash
# Create new project from official template
npx degit microsoft/PowerAppsCodeApps/templates/starter my-code-app
cd my-code-app

# Install dependencies
npm install

# Copy best practices configuration
# Download this repository and copy .github/ folder
# Or clone and copy:
git clone https://github.com/yourusername/power-apps-code-apps-best-practices.git temp-best-practices
cp -r temp-best-practices/.github/* .github/
rm -rf temp-best-practices

# Install git hooks dependencies
npm install --save-dev husky lint-staged prettier \
  @typescript-eslint/parser @typescript-eslint/eslint-plugin \
  eslint-plugin-react-hooks eslint-plugin-react-refresh

# Initialize husky
npm run prepare

# Verify setup
npm run dev  # Should start on port 3000
```

### Option 2: Existing Project

```bash
# 1. Navigate to your project
cd your-existing-code-app

# 2. Copy configuration files
# Download this repo and copy .github/ folder contents
# Or use curl/wget to download specific files

# 3. Install required dev dependencies
npm install --save-dev husky lint-staged prettier \
  @typescript-eslint/parser @typescript-eslint/eslint-plugin \
  eslint-plugin-react-hooks eslint-plugin-react-refresh

# 4. Add scripts to package.json (see below)

# 5. Initialize husky
npm run prepare
```

## Package.json Scripts Configuration

Add these scripts to your `package.json`:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "eslint . --ext ts,tsx --max-warnings 0",
    "lint:fix": "eslint . --ext ts,tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,json,css,md}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,json,css,md}\"",
    "typecheck": "tsc --noEmit",
    "validate": "npm run typecheck && npm run lint && npm run format:check",
    "preview": "vite preview",
    "prepare": "husky install"
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix --max-warnings 0",
      "prettier --write"
    ],
    "*.{json,css,md}": [
      "prettier --write"
    ]
  }
}
```

## Configuration Files

### 1. ESLint Configuration (eslint.config.js)

Copy `.github/husky-hooks/eslint.config.js` to your project root:

```bash
cp .github/husky-hooks/eslint.config.js .
```

Key features:
- Ignores `dist`, `node_modules`, `src/generated`
- TypeScript strict rules
- React hooks exhaustive-deps
- Power Apps specific rules (`no-explicit-any`, `no-unused-vars`, `no-console`)

### 2. Prettier Configuration (.prettierrc)

Copy `.github/husky-hooks/.prettierrc` to your project root:

```bash
cp .github/husky-hooks/.prettierrc .
```

### 3. Git Hooks

Copy hooks to `.husky/` directory:

```bash
mkdir -p .husky
cp .github/husky-hooks/pre-commit .husky/
cp .github/husky-hooks/pre-push .husky/
chmod +x .husky/pre-commit .husky/pre-push
```

## Git Hooks Details

### Pre-commit Hook (`.husky/pre-commit`)

Runs on every `git commit`:

1. **lint-staged** - ESLint + Prettier on staged files
2. **Code Apps Validations**:
   - `verbatimModuleSyntax: false` in tsconfig.json
   - `base: "./"` in vite.config.ts
   - Port 3000 in dev script
   - PowerProvider.tsx exists
   - src/generated folder exists
3. **Build validation** - `npm run build`

### Pre-push Hook (`.husky/pre-push`)

Runs on every `git push`:

1. **Full validation** - `npm run validate` (typecheck + lint + format:check)
2. **PAC CLI check** - Verifies authentication status

## VS Code Integration

### Recommended Extensions

```json
// .vscode/extensions.json
{
  "recommendations": [
    "ms-powerplatform.powerplatform-tools",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "bradlc.vscode-tailwindcss",
    "github.copilot",
    "github.copilot-chat"
  ]
}
```

### Settings (Optional)

```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "eslint.validate": ["typescript", "typescriptreact"],
  "prettier.enable": true
}
```

## PAC CLI Setup

```bash
# 1. Authenticate with your environment
pac auth create --environment <ENVIRONMENT_ID>

# 2. List available environments
pac env list

# 3. Select target environment
pac env select --environment <ENVIRONMENT_URL>

# 4. Initialize code app (if not done)
pac code init --displayName "My Code App"

# 5. Add data sources (connectors)
pac code add-data-source -a <API_NAME> -c <CONNECTION_ID>

# 6. Deploy
npm run build
pac code push
```

## GitHub Actions CI/CD

Copy `.github/workflows/validate.yml` to your repository:

```bash
mkdir -p .github/workflows
cp .github/workflows/validate.yml .github/workflows/
```

### Required Secrets (for deployment)

Add these secrets in GitHub Repository Settings → Secrets and variables → Actions:

| Secret | Description |
|--------|-------------|
| `PPAC_APP_ID` | Azure AD App Registration Client ID |
| `PPAC_CLIENT_SECRET` | Client Secret |
| `PPAC_TENANT_ID` | Azure AD Tenant ID |
| `PPAC_ENVIRONMENT_URL` | Power Platform Environment URL |

## Verification Checklist

After setup, verify everything works:

```bash
# 1. Check hooks are installed
ls -la .git/hooks/

# 2. Test pre-commit manually
npx lint-staged

# 3. Run full validation
npm run validate

# 4. Test build
npm run build

# 4. Start dev server
npm run dev
# Should show: "Local: http://localhost:3000/"

# 5. Make a test commit
git add .
git commit -m "test: verify hooks work"
# Should run all pre-commit checks

# 6. Test push (dry-run)
git push --dry-run
```

## Troubleshooting

### Hooks Not Running

```bash
# Reinstall husky
npm run prepare
# or
npx husky install
chmod +x .husky/pre-commit .husky/pre-push
```

### ESLint Errors in Generated Files

Add to `.eslintignore`:
```
src/generated/
dist/
node_modules/
```

### TypeScript Errors

```bash
# Check tsconfig.json has correct settings
cat tsconfig.json | grep verbatimModuleSyntax
# Should show: "verbatimModuleSyntax": false
```

### Port 3000 Already in Use

```bash
# Find and kill process
netstat -ano | findstr :3000
taskkill /PID <PID> /F
```

## Next Steps

1. Read [BEST_PRACTICES.md](BEST_PRACTICES.md) for development standards
2. Review [GIT_HOOKS.md](GIT_HOOKS.md) for hook customization
3. Configure [CI_CD.md](CI_CD.md) for your deployment needs
4. Reference [PAC_CLI.md](PAC_CLI.md) for CLI commands