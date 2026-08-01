# Contributing to Power Apps Code Apps Best Practices

Thank you for your interest in contributing! This repository provides best practices, configurations, and tooling for Microsoft Power Apps Code Apps development.

## How to Contribute

### Reporting Issues

1. **Check existing issues** first to avoid duplicates
2. **Use issue templates** when available
3. **Provide detailed information**:
   - Node.js version
   - PAC CLI version
   - Package.json dependencies
   - Error messages
   - Steps to reproduce
   - Expected vs actual behavior

### Suggesting Enhancements

1. Open an issue with the **enhancement** label
2. Describe the use case and benefit
3. Provide examples if possible
4. Reference Microsoft official documentation

### Pull Requests

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/your-feature-name`
3. **Make your changes** following the guidelines below
4. **Test your changes** locally
5. **Submit a pull request** with a clear description

## Development Guidelines

### Code Style

- Follow the existing code style in the repository
- Use **TypeScript** for all new code
- Follow **ESLint** and **Prettier** configurations
- Write **clear, descriptive commit messages**

### Documentation

- Update relevant documentation for any changes
- Keep README.md current
- Document new configuration options
- Include examples for new features

### Testing

- Test configuration changes in a real Code Apps project
- Verify hooks work correctly
- Test CI/CD pipeline changes
- Check cross-platform compatibility (Windows/macOS/Linux)

## Repository Structure

```
.github/
├── copilot-instructions.md           # GitHub Copilot config
├── agents/                           # AI agent definitions
├── instructions/                     # Development standards
├── skills/                           # Copilot skills
├── plugins/                          # Copilot plugins
├── husky-hooks/                      # Git hooks
├── workflows/                        # GitHub Actions
└── power-apps-code-apps-precommit-hooks.md

docs/
├── SETUP.md                          # Installation guide
├── BEST_PRACTICES.md                 # Development standards
├── GIT_HOOKS.md                      # Git hooks reference
├── CI_CD.md                          # CI/CD pipeline guide
├── PAC_CLI.md                        # PAC CLI reference
└── TROUBLESHOOTING.md                # Common issues
```

## Types of Contributions

### 1. Configuration Improvements
- ESLint rules for Code Apps
- Prettier configuration
- TypeScript settings
- Vite configuration

### 2. Git Hooks Enhancements
- New validation checks
- Performance improvements
- Better error messages
- Cross-platform compatibility

### 3. CI/CD Pipeline
- New workflow templates
- Environment configurations
- Deployment strategies
- Security improvements

### 4. Documentation
- Setup guides
- Best practices
- Troubleshooting
- API references

### 5. PAC CLI Integration
- New command references
- Workflow examples
- Troubleshooting tips

## Code of Conduct

This project follows the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).

### Our Standards

- **Be respectful** and inclusive
- **Welcome newcomers** and help them learn
- **Focus on constructive feedback**
- **Accept responsibility** for mistakes

### Unacceptable Behavior

- Harassment or discrimination
- Trolling or inflammatory comments
- Personal attacks
- Publishing private information

## Review Process

1. **Automated checks** run on all PRs:
   - ESLint
   - Prettier
   - TypeScript compilation
   - Markdown linting

2. **Maintainer review** for:
   - Code quality
   - Documentation accuracy
   - Alignment with Microsoft guidelines
   - Backward compatibility

3. **Approval required** from at least one maintainer

## Release Process

1. Changes merged to `main`
2. Version bump (semantic versioning)
3. Changelog updated
4. GitHub release created
5. Documentation deployed

## Getting Help

- **Discussions**: Use GitHub Discussions for questions
- **Issues**: Report bugs or request features
- **Discord**: Join Power Platform community Discord
- **Microsoft Learn**: Official documentation

## Recognition

Contributors will be recognized in:
- README.md contributors section
- Release notes
- GitHub contributors graph

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

## Sources

The content in this repository is based on the following sources:

| Source | Description | Link |
|--------|-------------|------|
| **Awesome GitHub Copilot** | Community-driven Copilot extensions gallery | [awesome-copilot.github.com](https://awesome-copilot.github.com/) |
| **Microsoft Power Apps Code Apps** | Official SDK, templates, and samples | [github.com/microsoft/PowerAppsCodeApps](https://github.com/microsoft/PowerAppsCodeApps) |
| **Microsoft Learn - Code Apps** | Official documentation and architecture | [learn.microsoft.com](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/) |
| **Power Platform CLI** | PAC CLI reference and installation | [learn.microsoft.com](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference) |

---

**Thank you for contributing to better Power Apps Code Apps development!**