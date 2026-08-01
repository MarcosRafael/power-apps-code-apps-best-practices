# Troubleshooting - Power Apps Code Apps

Common issues and solutions for Power Apps Code Apps development.

## Development Issues

### Port 3000 Already in Use

**Error**: `Error: listen EADDRINUSE: address already in use :::3000`

**Solution**:
```bash
# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# macOS/Linux
lsof -ti:3000 | xargs kill -9

# Or change port in vite.config.ts (not recommended for Code Apps)
server: {
  port: 3001,  # Only if absolutely necessary
  strictPort: false
}
```

### Authentication Failures

**Error**: `Authentication failed` or `401 Unauthorized`

**Solutions**:
```bash
# 1. Re-authenticate
pac auth clear
pac auth create --environment <ENV_ID>

# 2. Check environment access
pac env list
pac org who

# 3. Verify service principal (CI/CD)
# Check secrets in GitHub/Azure DevOps
# Ensure app registration has Dynamics CRM user_impersonation permission
```

### Package Installation Failures

**Error**: `npm ERR! code ERESOLVE` or peer dependency conflicts

**Solutions**:
```bash
# Clear cache and reinstall
npm cache clean --force
rm -rf node_modules package-lock.json
npm install

# Use legacy peer deps if needed
npm install --legacy-peer-deps

# Check Node version
node --version  # Should be 18.x or 20.x
```

### TypeScript Compilation Errors

**Error**: `TS2307: Cannot find module` or type errors

**Solutions**:
```bash
# 1. Check tsconfig.json
# Must have: "verbatimModuleSyntax": false

# 2. Check path aliases
"paths": {
  "@/*": ["./src/*"]
}

# 3. Restart TypeScript server in VS Code
# Ctrl+Shift+P → "TypeScript: Restart TS Server"

# 4. Check generated files exist
ls src/generated/services/
ls src/generated/models/
```

### Vite Dev Server Issues

**Error**: `Failed to resolve import` or HMR not working

**Solutions**:
```bash
# 1. Check vite.config.ts
import powerApps from '@microsoft/power-apps-vite';
plugins: [react(), powerApps()]

# 2. Check base path
base: "./"

# 3. Clear Vite cache
rm -rf node_modules/.vite
npm run dev

# 4. Check @microsoft/power-apps-vite version
npm list @microsoft/power-apps-vite
```

## Build Issues

### Build Fails with "verbatimModuleSyntax"

**Error**: Build fails or runtime errors in Power Platform

**Solution**:
```json
// tsconfig.json
{
  "compilerOptions": {
    "verbatimModuleSyntax": false
  }
}
```

### Missing PowerProvider

**Error**: App doesn't initialize in Power Platform

**Solution**:
```tsx
// src/PowerProvider.tsx
import type { ReactNode } from "react";

export default function PowerProvider({ children }: { children: ReactNode }) {
  return <>{children}</>;
}

// src/main.tsx
import PowerProvider from './PowerProvider';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <PowerProvider>
      <App />
    </PowerProvider>
  </StrictMode>
);
```

### Generated Files Missing

**Error**: `Cannot find module '@/generated/services/DataverseService'`

**Solution**:
```bash
# 1. Add data source
pac code add-data-source -a dataverse -c <CONNECTION_ID>

# 2. Check generated folder
ls src/generated/services/
ls src/generated/models/

# 3. Restart TypeScript server
# Ctrl+Shift+P → "TypeScript: Restart TS Server"
```

## Deployment Issues

### PAC CLI Push Fails

**Error**: `pac code push` fails with various errors

**Solutions**:
```bash
# 1. Build first
npm run build
pac code push

# 2. Check authentication
pac auth list
pac auth who

# 3. Verify environment
pac env list
pac env who

# 4. Debug mode
pac --debug code push

# 5. Common fixes:
# - Ensure solution exists (if using solutions)
# - Check app name uniqueness
# - Verify environment has Code Apps enabled
```

### Service Principal Authentication Fails

**Error**: `AADSTS7000215` or `Invalid client secret`

**Solutions**:
```bash
# 1. Verify secrets in GitHub/Azure DevOps
# PPAC_APP_ID, PPAC_CLIENT_SECRET, PPAC_TENANT_ID, PPAC_ENVIRONMENT_URL

# 2. Check app registration
# - Client secret not expired
# - Correct tenant ID
# - API permissions granted (Dynamics CRM user_impersonation)
# - Admin consent granted

# 3. Test locally
pac auth create \
  --applicationId $APP_ID \
  --clientSecret $SECRET \
  --tenant $TENANT \
  --environment $ENV_URL
```

### Environment Mismatch

**Error**: `Environment not found` or wrong environment

**Solutions**:
```bash
# 1. List environments
pac env list

# 2. Select correct environment
pac env select --environment <ENV_URL>

# 3. Verify in CI/CD
# Use correct PPAC_ENVIRONMENT_URL secret per environment
```

## Runtime Issues

### "App Timed Out" Error

**Error**: App shows "App timed out" in Power Platform

**Solutions**:
```bash
# 1. Ensure build was run
npm run build
pac code push

# 2. Check dist/ folder exists
ls dist/

# 3. Check index.html in dist/
cat dist/index.html

# 4. Verify base path in vite.config.ts
base: "./"
```

### Connector Authentication Prompts

**Error**: Users see consent dialogs repeatedly

**Solutions**:
```bash
# 1. Admin consent
# Go to Azure Portal → App Registration → API permissions → Grant admin consent

# 2. Suppress consent (for Microsoft connectors)
# In Power Platform Admin Center → Environment → Settings → Consent

# 3. Check connection references in solution
pac solution list
```

### Data Loading Failures

**Error**: Connector calls fail with 401/403

**Solutions**:
```typescript
// 1. Check connection exists
pac connection list

// 2. Verify connector added
pac code list-data-sources

// 3. Check permissions in Power Platform
// Environment → Settings → Users + permissions → Security roles

// 4. Handle errors in code
try {
  const data = await DataverseService.accounts.getAll();
} catch (error) {
  if (error.status === 401) {
    // Redirect to login or refresh token
  }
}
```

### UI Rendering Issues

**Error**: Components not displaying correctly

**Solutions**:
```bash
# 1. Check Fluent UI version compatibility
npm list @fluentui/react-components

# 2. Verify CSS imports
import '@fluentui/react-components/assets/fonts.css';

# 3. Check Tailwind/PostCSS config
# tailwind.config.js content paths
content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"]
```

## Git Hooks Issues

### Hooks Not Running

**Solutions**:
```bash
# 1. Check hooks exist
ls -la .git/hooks/

# 2. Reinstall
npm run prepare
# or
npx husky install
chmod +x .husky/pre-commit .husky/pre-push

# 3. Check package.json has prepare script
"prepare": "husky install"
```

### Lint-staged Not Finding Files

**Solutions**:
```bash
# 1. Check git status
git status

# 2. Stage files first
git add src/components/MyComponent.tsx

# 3. Check lint-staged config in package.json
"lint-staged": {
  "*.{ts,tsx}": ["eslint --fix", "prettier --write"]
}
```

### ESLint Errors in Generated Files

**Solution**:
```bash
# Add to .eslintignore
src/generated/
dist/
node_modules/
```

## Performance Issues

### Slow Build Times

**Solutions**:
```bash
# 1. Use Vite's built-in optimizations
# vite.config.ts
build: {
  minify: 'esbuild',  # Faster than terser
  cssCodeSplit: true,
}

# 2. Check bundle size
npm run build -- --report

# 3. Lazy load heavy components
const HeavyComponent = lazy(() => import('./HeavyComponent'));
```

### Large Bundle Size

**Solutions**:
```bash
# 1. Analyze bundle
npm run build -- --report

# 2. Code splitting
# 3. Remove unused dependencies
# 4. Use dynamic imports for heavy libraries
```

## VS Code Issues

### TypeScript Errors Not Showing

**Solutions**:
```bash
# 1. Restart TS Server
Ctrl+Shift+P → "TypeScript: Restart TS Server"

# 2. Check workspace version
# Use workspace TypeScript version
"typescript.tsdk": "node_modules/typescript/lib"

# 3. Reload window
Ctrl+Shift+P → "Developer: Reload Window"
```

### IntelliSense Not Working for Generated Files

**Solutions**:
```bash
# 1. Restart TS Server
# 2. Check tsconfig.json includes src/generated
"include": ["src", "src/generated"]

# 3. Regenerate types
pac code add-data-source -a dataverse -c <CONN_ID>
```

## Getting Help

### Official Resources

- [Power Apps Code Apps Docs](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/)
- [GitHub Repository](https://github.com/microsoft/PowerAppsCodeApps)
- [PAC CLI Reference](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference)
- [Power Platform Community](https://powerusers.microsoft.com/)

### Debugging Steps

1. **Check Node version** - Must be 18.x or 20.x LTS
2. **Clear caches** - `npm cache clean --force`, remove `node_modules/.vite`
3. **Reinstall dependencies** - `rm -rf node_modules package-lock.json && npm install`
4. **Verify configuration** - tsconfig.json, vite.config.ts, package.json
5. **Check PAC CLI version** - `pac --version`
6. **Enable debug logging** - `pac --debug code push`

### Reporting Issues

1. Check [existing issues](https://github.com/microsoft/PowerAppsCodeApps/issues)
2. Create minimal reproduction
3. Include:
   - Node version
   - PAC CLI version
   - Package.json dependencies
   - Error messages
   - Steps to reproduce