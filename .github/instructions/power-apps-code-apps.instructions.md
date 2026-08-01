---
description: 'Power Apps Code Apps development standards and best practices for TypeScript, React, and Power Platform integration'
applyTo: '**/*.{ts,tsx,js,jsx}, **/vite.config.*, **/package.json, **/tsconfig.json, **/power.config.json'
---

# Power Apps Code Apps Development Instructions

Instructions for generating high-quality Power Apps Code Apps using TypeScript, React, and Power Platform SDK, following Microsoft's official best practices and preview capabilities.

## Project Context

- **Power Apps Code Apps**: Code-first web app development with Power Platform integration
- **TypeScript + React**: Recommended frontend stack with Vite bundler
- **Power Platform SDK**: @microsoft/power-apps (current version ^1.0.3) for connector integration
- **PAC CLI**: Power Platform CLI for project management and deployment
- **Port 3000**: Required for local development with Power Platform SDK
- **Power Apps Premium**: End-user licensing requirement for production use

## Development Standards

### Project Structure
- Use well-organized folder structure with clear separation of concerns:
  ```
  src/
  ├── components/          # Reusable UI components
  ├── hooks/              # Custom React hooks for Power Platform
  ├── generated/
  │   ├── services/       # Generated connector services (PAC CLI)
  │   └── models/         # Generated TypeScript models (PAC CLI)
  ├── utils/             # Utility functions and helpers
  ├── types/             # TypeScript type definitions
  ├── PowerProvider.tsx  # Power Platform context wrapper
  └── main.tsx          # Application entry point
  ```
- Keep generated files (`generated/services/`, `generated/models/`) separate from custom code
- Use consistent naming conventions (kebab-case for files, PascalCase for components)

### TypeScript Configuration
- Set `verbatimModuleSyntax: false` in tsconfig.json for Power Apps SDK compatibility
- Enable strict mode for type safety
- Use proper typing for Power Platform connector responses
- Configure path alias with `"@": path.resolve(__dirname, "./src")` for cleaner imports
- Define interfaces for app-specific data structures
- Implement error boundaries and proper error handling types

### Advanced Power Platform Integration
#### Custom Control Frameworks (PCF Controls)
- Integrate PCF controls in Code Apps with proper event handling and data binding
- Package and deploy PCF controls with Code Apps

#### Power BI Embedded Analytics
- Embed Power BI reports using powerbi-client-react
- Dynamic report filtering based on Code App context
- Report export functionality (PDF, Excel, images)

#### AI Builder Integration
- Cognitive services for form processing, object detection
- Prediction models for business insights
- Sentiment analysis and language detection

#### Power Virtual Agents Integration
- Chatbot embedding using botframework-webchat
- Context passing between Code App and chatbot
- Custom bot actions triggering Code App functions

### React Patterns
- Use functional components with hooks for all new development
- Implement proper loading and error states for connector operations
- Consider Fluent UI React components (as used in official samples)
- Use React Query or SWR for data fetching and caching
- Follow React best practices for component composition
- Implement responsive design with mobile-first approach
- Install key dependencies: @microsoft/power-apps, @fluentui/react-components, concurrently

### Data Management
- Store sensitive data in data sources, never in application code
- Use generated models for type-safe connector operations
- Implement proper data validation and sanitization
- Handle offline scenarios gracefully where possible
- Cache frequently accessed data appropriately

#### Advanced Dataverse Relationships
- Many-to-many relationships with junction tables
- Polymorphic lookups (Account or Contact)
- Complex relationship queries with $expand and $filter
- Relationship validation with business rules

### Performance Optimization
- Use React.memo and useMemo for expensive computations
- Implement code splitting and lazy loading
- Optimize bundle size with tree shaking
- Use efficient connector query patterns
- Implement proper pagination for large data sets

### Security Best Practices
- Never store secrets or sensitive configuration in code
- Use Power Platform's built-in authentication and authorization
- Implement proper input validation and sanitization
- Follow OWASP security guidelines
- Respect Power Platform data loss prevention policies

### Error Handling
- Implement comprehensive error boundaries in React
- Handle connector-specific errors gracefully
- Provide meaningful error messages to users
- Log errors appropriately without exposing sensitive information
- Implement retry logic for transient failures

### Testing Strategies
- Write unit tests for business logic and utilities
- Test React components with React Testing Library
- Mock Power Platform connectors in tests
- Implement integration tests for critical user flows
- Use TypeScript for better test safety

### Development Workflow
- Use PAC CLI for project initialization and connector management
- `npm run dev` with concurrently running vite and pac code run
- `npm run build` followed by `pac code push` for deployment

### Deployment and DevOps
#### Multi-Environment Deployment Pipelines
- Environment-specific configurations for dev/test/staging/prod
- Automated deployment with Azure DevOps or GitHub Actions
- Environment promotion from dev → test → staging → prod
- Rollback strategies on deployment failures
- Configuration management with Azure Key Vault

## Current Limitations and Workarounds
- Content Security Policy (CSP) not yet supported
- Storage SAS IP restrictions not supported
- No Power Platform Git integration
- Dataverse solutions support limited

## Best Practices Summary
1. Follow Microsoft's official documentation and best practices
2. Use TypeScript for type safety and better developer experience
3. Implement proper error handling and user feedback
4. Optimize for performance and user experience
5. Follow security best practices and Power Platform policies
6. Write maintainable, testable, and well-documented code
7. Use generated services and models from PAC CLI
8. Plan for future feature updates and migrations

## Sources

This instructions file is based on the **Power Apps Code Apps** instructions from [Awesome GitHub Copilot](https://awesome-copilot.github.com/instructions/power-apps-code-apps/), a community-driven collection of GitHub Copilot extensions. Content aligns with:

- [Microsoft Power Apps Code Apps Repository](https://github.com/microsoft/PowerAppsCodeApps) — Official SDK, samples (HelloWorld, FluentSample, StaticAssetTracker), and templates (starter, vite)
- [Microsoft Learn - Code Apps Documentation](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/) — Development standards, architecture, and system limits
- [Power Apps SDK (@microsoft/power-apps)](https://www.npmjs.com/package/@microsoft/power-apps) — npm package documentation
- [Fluent UI React](https://react.fluentui.dev/) — Microsoft's React component library used in official samples
- [Vite](https://vite.dev/) — Build tool used by Code Apps templates