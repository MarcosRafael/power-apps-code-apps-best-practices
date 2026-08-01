# GitHub Copilot Instructions - Power Apps Code Apps

> **Purpose**: This file configures **GitHub Copilot in Visual Studio Code** for Power Apps Code Apps development.

This project uses GitHub Copilot with specialized configurations for **Power Apps Code Apps** development.

## Available Resources

### Agent
- **Power Platform Expert** (`.github/agents/power-platform-expert.agent.md`) — Expert guidance on Power Apps Code Apps, PAC CLI, Power Platform SDK, connector integration, and deployment.

### Instructions
- **Power Apps Code Apps** (`.github/instructions/power-apps-code-apps.instructions.md`) — Development standards for TypeScript/React Code Apps with Power Platform SDK integration. **Auto-applied to**: `**/*.{ts,tsx,js,jsx}`, `**/vite.config.*`, `**/package.json`, `**/tsconfig.json`, `**/power.config.json`

### Skill
- **Power Apps Code App Scaffold** (`.github/skills/power-apps-code-app-scaffold/SKILL.md`) — Scaffold complete Code Apps projects with PAC CLI setup.

### Plugin
- **Power Apps Code Apps** (`.github/plugins/power-apps-code-apps/README.md`) — Complete toolkit for Code Apps development.

### Validation & Quality Gates
- **Pre-commit Hooks** (`.github/power-apps-code-apps-precommit-hooks.md`) — Comprehensive git hooks configuration with Husky + lint-staged for Code Apps
- **Husky Hooks** (`.github/husky-hooks/`) — Ready-to-use pre-commit and pre-push hooks
- **GitHub Actions** (`.github/workflows/validate.yml`) — CI/CD pipeline with Code Apps validation

## How to Use in VS Code

1. **Clone this repository** or copy the `.github/` folder to your Power Apps Code Apps project root
2. **Open in VS Code** - GitHub Copilot will automatically detect and use these configurations
3. **Copilot will auto-apply** instructions based on file patterns (`applyTo` in instructions)
4. **Use the Agent** - Type `@power-platform-expert` in Copilot Chat for specialized guidance
5. **Use the Skill** - Type `/power-apps-code-app-scaffold` in Copilot Chat to scaffold projects

## Sources

- **[Awesome GitHub Copilot](https://awesome-copilot.github.com/)** — Community-driven collection of GitHub Copilot extensions. Source for the agent, instructions, skill, and plugin configurations.
- **[Microsoft Power Apps Code Apps Repository](https://github.com/microsoft/PowerAppsCodeApps)** — Official Microsoft samples, templates, and SDK for Power Apps Code Apps.
- **[Microsoft Learn - Code Apps Documentation](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/)** — Official documentation for development standards, architecture, and deployment.