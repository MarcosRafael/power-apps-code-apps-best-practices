# Best Practices - Power Apps Code Apps

Comprehensive development standards and patterns for building high-quality Power Apps Code Apps.

## Project Structure

```
src/
├── components/          # Reusable UI components
│   ├── ui/             # Base UI components (Button, Input, Card, etc.)
│   ├── layout/         # Layout components (Header, Sidebar, Footer)
│   └── features/       # Feature-specific components
├── hooks/              # Custom React hooks for Power Platform
│   ├── usePowerPlatform.ts
│   ├── useConnector.ts
│   └── useAuth.ts
├── generated/          # PAC CLI generated files (DO NOT EDIT MANUALLY)
│   ├── services/       # Connector service clients
│   └── models/         # TypeScript interfaces/types
├── utils/              # Utility functions
│   ├── helpers.ts
│   ├── constants.ts
│   └── validators.ts
├── types/              # Application-specific type definitions
│   ├── app.types.ts
│   └── api.types.ts
├── PowerProvider.tsx   # Power Platform context provider
└── main.tsx           # Application entry point
```

### Key Principles

1. **Separation of Concerns** - Keep generated code separate from custom code
2. **Colocation** - Keep related files together
3. **Barrel Exports** - Use index.ts for clean imports
4. **Naming Conventions** - kebab-case for files, PascalCase for components

## TypeScript Configuration

### Required tsconfig.json Settings

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

### Critical Settings Explained

| Setting | Value | Reason |
|---------|-------|--------|
| `verbatimModuleSyntax` | `false` | Required for Power Apps SDK compatibility |
| `strict` | `true` | Enables all strict type checking |
| `noUnusedLocals` | `true` | Catches unused variables |
| `baseUrl` + `paths` | `@/*` | Clean import aliases |

## Vite Configuration

### Required vite.config.ts

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

### Critical Settings

| Setting | Value | Reason |
|---------|-------|--------|
| `base` | `"./"` | Required for Power Platform deployment |
| `server.port` | `3000` | Required by Power Apps SDK |
| `plugins` | `powerApps()` | Official Power Apps Vite plugin |

## PowerProvider Pattern

### Implementation (src/PowerProvider.tsx)

```tsx
import type { ReactNode } from "react";

export default function PowerProvider({ children }: { children: ReactNode }) {
  // Power Apps SDK v1.0+ doesn't require explicit initialization
  // The SDK auto-initializes when running in Power Platform
  // For local development, the Vite plugin handles mocking
  
  return <>{children}</>;
}
```

### Usage in main.tsx

```tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import PowerProvider from './PowerProvider';
import App from './App';
import './index.css';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <PowerProvider>
      <App />
    </PowerProvider>
  </StrictMode>
);
```

## Connector Integration

### Adding Data Sources

```bash
# List available connections
pac connection list

# Add SQL Server connector
pac code add-data-source -a sql -c <CONNECTION_ID>

# Add Dataverse connector
pac code add-data-source -a dataverse -c <CONNECTION_ID>

# Add Office 365 Users
pac code add-data-source -a office365users -c <CONNECTION_ID>
```

### Using Generated Services

```typescript
// Generated in src/generated/services/
import { SqlService, DataverseService, Office365UsersService } from '@/generated/services';

// Type-safe connector operations
const accounts = await DataverseService.accounts.getAll({
  select: ['name', 'accountid', 'revenue'],
  filter: "statecode eq 0",
  top: 100
});

const currentUser = await Office365UsersService.MyProfile_V2(
  "id,displayName,mail,userPrincipalName"
);
```

### Error Handling Pattern

```typescript
import { useState } from 'react';
import { DataverseService } from '@/generated/services';

export function useAccounts() {
  const [accounts, setAccounts] = useState<Account[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const fetchAccounts = async () => {
    setLoading(true);
    setError(null);
    try {
      const result = await DataverseService.accounts.getAll({
        select: ['name', 'accountid', 'revenue'],
        top: 50
      });
      setAccounts(result.value);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  };

  return { accounts, loading, error, fetchAccounts };
}
```

## React Patterns

### Component Structure

```tsx
// components/features/AccountList/AccountList.tsx
import { useAccounts } from '@/hooks/useAccounts';
import { AccountCard } from './AccountCard';
import { LoadingSpinner } from '@/components/ui/LoadingSpinner';
import { ErrorMessage } from '@/components/ui/ErrorMessage';

export function AccountList() {
  const { accounts, loading, error, fetchAccounts } = useAccounts();

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} onRetry={fetchAccounts} />;

  return (
    <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
      {accounts.map(account => (
        <AccountCard key={account.accountid} account={account} />
      ))}
    </div>
  );
}
```

### Custom Hooks for Power Platform

```typescript
// hooks/useConnector.ts
import { useCallback, useState } from 'react';

export function useConnector<T>(
  connectorCall: () => Promise<T>
) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const execute = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const result = await connectorCall();
      setData(result);
      return result;
    } catch (err) {
      const error = err instanceof Error ? err : new Error('Unknown error');
      setError(error);
      throw error;
    } finally {
      setLoading(false);
    }
  }, [connectorCall]);

  return { data, loading, error, execute };
}
```

## Data Management

### Type-Safe Models

```typescript
// types/dataverse.types.ts
export interface Account {
  accountid: string;
  name: string;
  revenue?: number;
  statecode: number;
  createdon: string;
  modifiedon: string;
}

export interface Contact {
  contactid: string;
  fullname: string;
  emailaddress1?: string;
  telephone1?: string;
  parentcustomerid_account?: Account;
}
```

### Pagination Pattern

```typescript
export async function fetchAllPages<T>(
  fetchPage: (skip: number) => Promise<{ value: T[]; '@odata.nextLink'?: string }>
): Promise<T[]> {
  const allItems: T[] = [];
  let skip = 0;
  let hasMore = true;

  while (hasMore) {
    const { value, '@odata.nextLink': nextLink } = await fetchPage(skip);
    allItems.push(...value);
    hasMore = !!nextLink;
    skip += value.length;
  }

  return allItems;
}
```

## Performance Optimization

### Code Splitting

```tsx
// Lazy load heavy components
const HeavyChart = lazy(() => import('./HeavyChart'));
const ReportViewer = lazy(() => import('./ReportViewer'));

// In component
<Suspense fallback={<LoadingSpinner />}>
  <HeavyChart data={data} />
</Suspense>
```

### Memoization

```tsx
// Memoize expensive computations
const sortedAccounts = useMemo(
  () => accounts.toSorted((a, b) => b.revenue - a.revenue),
  [accounts]
);

// Memoize callbacks
const handleSelect = useCallback((id: string) => {
  navigate(`/accounts/${id}`);
}, [navigate]);

// Memoize components
const AccountCard = memo(function AccountCard({ account }) {
  return <div>{account.name}</div>;
});
```

## Security Best Practices

### Environment Variables

```bash
# .env.local (NOT committed)
VITE_API_BASE_URL=https://api.example.com
VITE_APP_INSIGHTS_KEY=your-key

# .env.example (committed)
VITE_API_BASE_URL=
VITE_APP_INSIGHTS_KEY=
```

```typescript
// Access in code
const apiUrl = import.meta.env.VITE_API_BASE_URL;
```

### Never Store Secrets

```typescript
// ❌ BAD - Never do this
const apiKey = 'sk-1234567890abcdef';

// ✅ GOOD - Use Power Platform connections
const result = await SqlService.executeQuery({
  query: "SELECT * FROM Users WHERE Id = @id",
  parameters: { id: userId }
});
```

### Input Validation

```typescript
import { z } from 'zod';

const AccountSchema = z.object({
  name: z.string().min(1).max(100),
  revenue: z.number().min(0).optional(),
  email: z.string().email().optional(),
});

export function validateAccount(data: unknown) {
  return AccountSchema.safeParse(data);
}
```

## Testing

### Unit Tests

```typescript
// hooks/__tests__/useAccounts.test.ts
import { renderHook, act } from '@testing-library/react';
import { useAccounts } from '../useAccounts';

vi.mock('@/generated/services', () => ({
  DataverseService: {
    accounts: {
      getAll: vi.fn().mockResolvedValue({
        value: [{ accountid: '1', name: 'Test Account', revenue: 1000 }]
      })
    }
  }
}));

test('fetches accounts successfully', async () => {
  const { result } = renderHook(() => useAccounts());
  
  await act(async () => {
    await result.current.fetchAccounts();
  });
  
  expect(result.current.accounts).toHaveLength(1);
  expect(result.current.loading).toBe(false);
});
```

### Integration Tests

```typescript
// e2e/account-flow.spec.ts
import { test, expect } from '@playwright/test';

test('user can view account list', async ({ page }) => {
  await page.goto('/accounts');
  await expect(page.locator('text=Accounts')).toBeVisible();
  await expect(page.locator('[data-testid=account-card]')).toHaveCount(10);
});
```

## Deployment

### Build Process

```bash
# 1. Type check
npm run typecheck

# 2. Lint
npm run lint

# 3. Format check
npm run format:check

# 4. Build
npm run build
# Output: dist/ folder

# 5. Deploy
pac code push
```

### Multi-Environment

```bash
# Development
pac auth create --environment <DEV_ENV_ID>
pac code push

# Test
pac auth create --environment <TEST_ENV_ID>
pac code push

# Production
pac auth create --environment <PROD_ENV_ID>
pac code push
```

## Common Pitfalls to Avoid

| ❌ Don't | ✅ Do |
|----------|-------|
| Edit `src/generated/` manually | Run `pac code add-data-source` to regenerate |
| Use `verbatimModuleSyntax: true` | Keep `false` for SDK compatibility |
| Hardcode API URLs | Use Power Platform connections |
| Store secrets in code | Use environment variables / Key Vault |
| Skip error boundaries | Wrap connector calls in try/catch |
| Ignore TypeScript errors | Fix all errors before commit |
| Use `any` type | Define proper interfaces |
| Commit without hooks | Run `npm run prepare` after clone |

## References

- [Power Apps Code Apps Docs](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/)
- [Official Samples](https://github.com/microsoft/PowerAppsCodeApps/tree/main/samples)
- [PAC CLI Reference](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/code)
- [Power Apps SDK](https://www.npmjs.com/package/@microsoft/power-apps)