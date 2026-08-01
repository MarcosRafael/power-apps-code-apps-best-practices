# Power Apps Code Apps - Pre-commit Hooks Configuration

This document provides a complete pre-commit hook setup for Power Apps Code Apps projects to enforce best practices and prevent bad patterns before code is committed.

## Quick Setup

```bash
# Install dependencies
npm install --save-dev husky lint-staged prettier @typescript-eslint/parser @typescript-eslint/eslint-plugin

# Initialize husky
npx husky install

# Add pre-commit hook
npx husky add .husky/pre-commit "npx lint-staged"
```

> **Note**: The configurations in this document are based on Power Apps SDK requirements from [Microsoft's official documentation](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/) and the [Awesome GitHub Copilot](https://awesome-copilot.github.com/) community project.

## Package.json Configuration

Add to your `package.json`:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
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
    ],
    "*.{ts,tsx,json}": [
      "tsc --noEmit --skipLibCheck"
    ]
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "pre-push": "npm run validate"
    }
  }
}
```

## Power Apps Code Apps Specific Validations

Create `.husky/pre-commit` with custom validations:

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

## ESLint Configuration for Code Apps

Create `eslint.config.js` (flat config):

```javascript
import js from '@eslint/js';
import typescriptEslint from 'typescript-eslint';
import reactHooks from 'eslint-plugin-react-hooks';
import reactRefresh from 'eslint-plugin-react-refresh';

export default typescriptEslint.config(
  { ignores: ['dist', 'node_modules', 'src/generated'] },
  {
    extends: [
      js.configs.recommended,
      ...typescriptEslint.configs.recommended,
      ...typescriptEslint.configs.stylistic,
    ],
    files: ['**/*.{ts,tsx}'],
    languageOptions: {
      ecmaVersion: 2020,
      globals: { browser: true, es2020: true },
      parserOptions: {
        project: ['./tsconfig.json', './tsconfig.app.json'],
        tsconfigRootDir: import.meta.dirname,
      },
    },
    settings: { react: { version: '18.3' } },
    plugins: {
      'react-hooks': reactHooks,
      'react-refresh': reactRefresh,
    },
    rules: {
      ...reactHooks.configs.recommended.rules,
      'react-refresh/only-export-components': [
        'warn',
        { allowConstantExport: true },
      ],
      // Power Apps Code Apps specific rules
      '@typescript-eslint/no-explicit-any': 'warn',
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      'no-console': ['warn', { allow: ['warn', 'error'] }],
    },
  }
);
```

## Prettier Configuration

Create `.prettierrc`:

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}
```

## TypeScript Configuration (tsconfig.json)

Ensure these settings for Code Apps compatibility:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "verbatimModuleSyntax": false,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

## Vite Configuration (vite.config.ts)

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';
import powerApps from '@microsoft/power-apps-vite';

export default defineConfig({
  plugins: [react(), powerApps()],
  base: "./",
  server: {
    port: 3000,
    strictPort: true,
    host: true,
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
  },
});
```

## Pre-push Hook (Optional - runs full validation)

Create `.husky/pre-push`:

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

## Common Bad Practices These Hooks Prevent

| Bad Practice | Hook That Catches It |
|--------------|---------------------|
| `verbatimModuleSyntax: true` in tsconfig | Custom pre-commit check |
| Missing `base: "./"` in vite.config.ts | Custom pre-commit check |
| Console.log in production code | ESLint `no-console` rule |
| Unused variables/imports | TypeScript `noUnusedLocals` + ESLint |
| Type `any` usage | ESLint `@typescript-eslint/no-explicit-any` |
| Formatting inconsistencies | Prettier |
| Build failures | `npm run build` in pre-commit |
| Missing PowerProvider | Custom pre-commit check |
| Port not 3000 | Custom pre-commit check |

## CI/CD Integration (GitHub Actions)

Add to `.github/workflows/validate.yml`:

```yaml
name: Validate Code App

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run validate
      - run: npm run build
      - name: Check PAC CLI
        run: |
          npm install -g @microsoft/powerplatform-cli
          pac --version
```

## Troubleshooting

### Husky not running
```bash
# Reinstall husky
npm run prepare
# or
npx husky install
chmod +x .husky/pre-commit
```

### Lint-staged not finding files
```bash
# Check git status
git status
# Ensure files are staged
git add .
```

### TypeScript errors in generated files
```bash
# Add to .eslintignore and .prettierignore
src/generated/
```

## References

- [Power Apps Code Apps Documentation](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/)
- [Official Templates](https://github.com/microsoft/PowerAppsCodeApps/tree/main/templates)
- [Husky Documentation](https://typicode.github.io/husky/)
- [lint-staged Documentation](https://github.com/okonet/lint-staged)

## Sources

| Source | Description | Link |
|--------|-------------|------|
| **Microsoft Power Apps Code Apps** | Official SDK requirements and templates | [github.com/microsoft/PowerAppsCodeApps](https://github.com/microsoft/PowerAppsCodeApps) |
| **Awesome GitHub Copilot** | Community-driven Copilot extensions gallery | [awesome-copilot.github.com](https://awesome-copilot.github.com/) |
| **Husky** | Git hooks manager for Node.js | [typicode.github.io/husky](https://typicode.github.io/husky/) |
| **lint-staged** | Run linters on staged git files | [github.com/okonet/lint-staged](https://github.com/okonet/lint-staged) |
| **Power Apps SDK** | @microsoft/power-apps npm package | [npmjs.com/package/@microsoft/power-apps](https://www.npmjs.com/package/@microsoft/power-apps) |
| **Vite** | Frontend tooling used by Code Apps | [vite.dev](https://vite.dev/) |

### Validation Rules Attribution

- **`verbatimModuleSyntax: false` check** — Required by [Power Apps SDK](https://www.npmjs.com/package/@microsoft/power-apps) compatibility
- **`base: "./"` check** — Required for [Power Apps Code Apps deployment](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/)
- **Port 3000 check** — Required by [Microsoft starter templates](https://github.com/microsoft/PowerAppsCodeApps/tree/main/templates)
- **PowerProvider.tsx check** — Based on [official Code Apps samples](https://github.com/microsoft/PowerAppsCodeApps/tree/main/samples) pattern