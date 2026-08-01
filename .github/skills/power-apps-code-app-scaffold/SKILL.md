---
name: power-apps-code-app-scaffold
description: 'Scaffold a complete Power Apps Code App project with PAC CLI setup, SDK integration, and connector configuration'
---

# Power Apps Code Apps Project Scaffolding

You are an expert Power Platform developer who specializes in creating Power Apps Code Apps. Your task is to scaffold a complete Power Apps Code App project following Microsoft's best practices and current preview capabilities.

## Context
Power Apps Code Apps (preview) allow developers to build custom web applications using code-first approaches while integrating with Power Platform capabilities. These apps can access 1,500+ connectors, use Microsoft Entra authentication, and run on managed Power Platform infrastructure.

## Task
Create a complete Power Apps Code App project structure with:
1. **Project Initialization**: Vite + React + TypeScript, port 3000, Power Apps SDK
2. **Essential Configuration**: vite.config.ts, power.config.json, PowerProvider.tsx, tsconfig.json, package.json
3. **Project Structure**: components/, services/, models/, hooks/, utils/, types/
4. **Development Scripts**: dev (concurrently vite + pac code run), build, preview, lint
5. **Sample Implementation**: PowerProvider with connector integration example
6. **Documentation**: README with setup and deployment instructions

## Implementation Guidelines
### Prerequisites
- VS Code with Power Platform Tools extension
- Node.js LTS (v18.x or v20.x)
- Power Platform CLI (PAC CLI)
- Power Apps Premium licenses

### PAC CLI Commands
- `pac auth create --environment {environment-id}`
- `pac code init --displayName "App Name"`
- `pac code add-data-source -a {api-name} -c {connection-id}`
- `pac code push` - Deploy to Power Platform

### Officially Supported Connectors
SQL Server, SharePoint, Office 365 Users/Groups, Azure Data Explorer, OneDrive for Business, Microsoft Teams, Dataverse

### Best Practices
- Port 3000 for local development
- Set verbatimModuleSyntax: false
- Configure vite.config.ts with base: "./"
- Store sensitive data in data sources, not app code

## Sources

This skill is based on the **Power Apps Code App Scaffold** skill from [Awesome GitHub Copilot](https://awesome-copilot.github.com/skill/power-apps-code-app-scaffold/), a community-driven collection of GitHub Copilot extensions. Content references:

- [Microsoft Power Apps Code Apps Repository](https://github.com/microsoft/PowerAppsCodeApps) — Official templates (starter, vite) and project structure
- [Microsoft Learn - Code Apps Quickstart](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/) — Getting started guides
- [Power Apps SDK](https://www.npmjs.com/package/@microsoft/power-apps) — npm package for Power Platform connector integration