# Git Hooks Reference - Power Apps Code Apps

Complete reference for the pre-commit and pre-push hooks configuration.

## Overview

This project uses **Husky** + **lint-staged** to enforce code quality and Power Apps Code Apps specific requirements before commits and pushes.

## Hook Files

### Pre-commit Hook (`.husky/pre-commit`)

Runs on every `git commit`:

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

echo "🔍 Running Power Apps Code Apps pre-commit checks..."

# 1. Run lint-staged for staged files
npx lint-staged

# 2. Power Apps Code Apps specific validations
echo "📋 Validating Power Apps Code Apps configuration..."

# Check tsconfig.json for verbatimModuleSyntax: false
if ! grep -q '"verbatimModuleSyntax": false' tsconfig.json; then
  echo "❌ ERROR: tsconfig.json must have 'verbatimModuleSyntax: false' for Power Apps SDK compatibility"
  exit 1
fi

# Check vite.config.ts for base: "./"
if ! grep -q 'base: "./"' vite.config.ts; then
  echo "❌ ERROR: vite.config.ts must have 'base: \"./\"' for Power Apps deployment"
  exit 1
fi

# Check package.json for port 3000 in dev script
if ! grep -q '"dev": "vite"' package.json && ! grep -q '"dev": "concurrently' package.json; then
  echo "⚠️  WARNING: Ensure dev server runs on port 3000 (required by Power Apps SDK)"
fi

# Check for PowerProvider.tsx existence
if [ ! -f "src/PowerProvider.tsx" ]; then
  echo "⚠️  WARNING: PowerProvider.tsx not found - required for Power Platform initialization"
fi

# Check for generated folder structure
if [ ! -d "src/generated" ]; then
  echo "ℹ️  INFO: src/generated folder not found - run 'pac code add-data-source' to generate connector services"
fi

# 3. Validate build passes
echo "🏗️  Validating build..."
npm run build --if-present

echo "✅ All pre-commit checks passed!"
```

### Pre-push Hook (`.husky/pre-push`)

Runs on every `git push`:

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

echo "🚀 Running pre-push validation..."

# Full validation suite
npm run validate

# Check if PAC CLI is available and authenticated
if command -v pac &> /dev/null; then
  echo "🔐 Checking PAC CLI authentication..."
  pac auth list
else
  echo "⚠️  PAC CLI not found - install from https://aka.ms/PowerAppsCLI"
fi

echo "✅ Pre-push validation complete!"
```

## Package.json Configuration

### Scripts

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
  }
}
```

### lint-staged Configuration

```json
{
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

## What Each Hook Validates

### Pre-commit Validations

| Check | Tool | Failure Action |
|-------|------|----------------|
| ESLint errors | ESLint | Blocks commit |
| Prettier formatting | Prettier | Auto-fixes |
| TypeScript types | tsc --noEmit | Blocks commit |
| verbatimModuleSyntax: false | grep | Blocks commit |
| base: "./" in vite.config.ts | grep | Blocks commit |
| Port 3000 in dev script | grep | Warning only |
| PowerProvider.tsx exists | file check | Warning only |
| src/generated/ exists | dir check | Info only |
| Build succeeds | npm run build | Blocks commit |

### Pre-push Validations

| Check | Tool | Failure Action |
|-------|------|----------------|
| TypeScript types | tsc --noEmit | Blocks push |
| ESLint | eslint | Blocks push |
| Prettier | prettier --check | Blocks push |
| PAC CLI installed | command -v | Warning only |
| PAC CLI authenticated | pac auth list | Info only |

## Installation

```bash
# 1. Install dependencies
npm install --save-dev husky lint-staged prettier \
  @typescript-eslint/parser @typescript-eslint/eslint-plugin \
  eslint-plugin-react-hooks eslint-plugin-react-refresh

# 2. Initialize husky
npx husky install

# 3. Add hooks (or copy from .github/husky-hooks/)
npx husky add .husky/pre-commit "npx lint-staged"
npx husky add .husky/pre-push "npm run validate"

# 4. Make executable
chmod +x .husky/pre-commit .husky/pre-push

# 5. Add prepare script to package.json
# "prepare": "husky install"
```

## Customization

### Skip Hooks Temporarily

```bash
# Skip pre-commit
git commit --no-verify -m "message"

# Skip pre-push
git push --no-verify
```

### Modify Validation Rules

Edit `.husky/pre-commit` to customize:

```bash
# Example: Change port requirement
# Check for custom port
if ! grep -q 'port: 3000' vite.config.ts; then
  echo "⚠️  WARNING: vite.config.ts should have port 3000"
fi

# Example: Add environment variable check
if [ ! -f ".env.local" ]; then
  echo "⚠️  WARNING: .env.local not found"
fi

# Example: Check for specific connector
if [ ! -f "src/generated/services/DataverseService.ts" ]; then
  echo "ℹ️  INFO: Dataverse connector not configured"
fi
```

### Add Custom Lint Rules

Edit `.github/husky-hooks/eslint.config.js`:

```javascript
rules: {
  // ... existing rules
  
  // Custom rules for your project
  'no-restricted-imports': [
    'error',
    {
      paths: [{
        name: 'lodash',
        message: 'Use native JS methods instead of lodash'
      }]
    }
  ],
  
  // Enforce specific patterns
  '@typescript-eslint/consistent-type-imports': 'error',
  '@typescript-eslint/no-floating-promises': 'warn',
}
```

## Troubleshooting

### Hooks Not Running

```bash
# Check if hooks exist
ls -la .git/hooks/

# Reinstall
npm run prepare
# or
npx husky install
chmod +x .husky/pre-commit .husky/pre-push
```

### "husky: command not found"

```bash
# Install husky locally
npm install --save-dev husky

# Or use npx
npx husky install
```

### lint-staged Not Finding Files

```bash
# Check git status
git status

# Ensure files are staged
git add src/components/MyComponent.tsx
```

### ESLint Errors in Generated Files

Add to `.eslintignore`:
```
src/generated/
dist/
node_modules/
```

### Build Fails in Hook but Works Locally

```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install

# Check Node version matches
node --version  # Should be 18.x or 20.x
```

### Pre-push Takes Too Long

```bash
# Run validation in parallel (if independent)
# Or skip for quick pushes
git push --no-verify
```

## CI/CD Integration

The same validations run in GitHub Actions (`.github/workflows/validate.yml`):

```yaml
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
    # Same checks as pre-commit hook
```

## Best Practices

1. **Never skip hooks** in normal workflow - fix the issues instead
2. **Run `npm run validate`** before pushing to catch issues early
3. **Keep hooks fast** - avoid heavy operations in pre-commit
4. **Use `--no-verify` sparingly** - only for emergency fixes
5. **Update hooks** when project requirements change
6. **Document custom validations** for team members

## References

- [Husky Documentation](https://typicode.github.io/husky/)
- [lint-staged Documentation](https://github.com/okonet/lint-staged)
- [Power Apps Code Apps Requirements](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/)