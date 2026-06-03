# Contributing to NKP Harbor Deployment Guide

Thank you for your interest in contributing! This document provides guidelines and instructions for contributing to this project.

## Code of Conduct

We are committed to providing a welcoming and inclusive environment for all contributors. Please be respectful and constructive in all interactions.

## How to Contribute

### Reporting Issues

If you encounter bugs, issues, or have suggestions for improvements:

1. Check existing [Issues](../../issues) to avoid duplicates
2. Create a new issue with a clear title and description
3. Include relevant details:
   - NKP and Harbor versions used
   - Your environment (OS, Nutanix AHV configuration)
   - Steps to reproduce (if applicable)
   - Expected vs. actual behavior
   - Error messages or logs

### Feature Requests

We welcome feature suggestions! Please:

1. Open an issue with the label `enhancement`
2. Clearly describe the feature and its benefits
3. Explain use cases and implementation ideas if possible
4. Reference related issues or PRs if applicable

### Pull Requests

#### Before Starting

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Read through existing documentation to understand the structure

#### Development Guidelines

**For Documentation:**
- Follow markdown best practices
- Use clear, concise language
- Include code examples where applicable
- Test all commands before documenting
- Maintain consistency with existing style

**For Scripts:**
- Write bash scripts with proper error handling
- Include comments explaining complex logic
- Add usage examples and help text
- Test in an air-gapped environment if possible
- Follow naming conventions (e.g., `##-descriptive-name.sh`)

**For Configuration Files:**
- Validate YAML/configuration syntax
- Include comments explaining non-obvious settings
- Use placeholder values clearly marked (e.g., `<YOUR_VALUE>`)
- Document all configurable parameters

#### Submitting a Pull Request

1. Ensure your code/documentation is complete and tested
2. Commit with clear, descriptive messages:
   ```bash
   git commit -m "Add feature: Brief description of changes"
   ```
3. Push to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
4. Open a Pull Request with:
   - Clear title describing the change
   - Detailed description of what changed and why
   - Reference to related issues (e.g., "Fixes #123")
   - Any relevant testing notes

#### PR Review Process

- Maintainers will review your PR within 5-7 business days
- Feedback may be requested for improvements
- Once approved, your PR will be merged
- Thank you for contributing!

## Documentation Standards

### README Sections

When updating README.md:
- Keep section headings at appropriate levels (H2, H3)
- Use consistent formatting for code blocks
- Include practical examples
- Maintain table formatting consistency

### Code Examples

- All bash commands should be tested
- Provide context before command blocks
- Explain expected output
- Use clear variable names
- Include error handling examples

### Configuration Examples

- Mark required fields clearly
- Provide default values where applicable
- Document parameter purposes
- Include validation rules

## Testing Guidelines

Before submitting changes:

1. **Documentation Changes:**
   - Review markdown rendering locally
   - Verify all links are correct
   - Check code block syntax highlighting
   - Test all code examples if possible

2. **Script Changes:**
   - Test in relevant environment (Rocky/RHEL 9)
   - Verify error handling
   - Check for unintended side effects
   - Document any new dependencies

3. **Configuration Changes:**
   - Validate syntax (YAML, XML, etc.)
   - Test in non-production environment
   - Document any breaking changes

## Repository Structure

```
nkp-harbor-deployment/
├── README.md              # Main documentation
├── LICENSE                # MIT License
├── CONTRIBUTING.md        # This file
├── scripts/               # Automation scripts
│   ├── 01-*.sh           # Setup scripts
│   └── 05-*.sh           # Deployment scripts
├── configs/               # Configuration templates
│   ├── harbor.yml.template
│   ├── openssl.cnf
│   └── docker-compose.override.yml
├── docs/                  # Additional documentation
│   ├── ARCHITECTURE.md
│   ├── TROUBLESHOOTING.md
│   ├── SSL-CERTIFICATE-RENEWAL.md
│   └── CLUSTER-OPERATIONS.md
└── examples/              # Example configurations
    ├── cluster-config-example.yaml
    └── capi-machine-pool.yaml
```

When adding new files:
- Place scripts in `scripts/` with numbered prefixes
- Place templates in `configs/`
- Place guides in `docs/`
- Place examples in `examples/`

## Commit Message Guidelines

Use clear, descriptive commit messages:

```bash
# Good examples:
git commit -m "docs: Add Harbor certificate renewal guide"
git commit -m "scripts: Improve error handling in setup-bastion.sh"
git commit -m "fix: Correct storage mount documentation"
git commit -m "feature: Add CAPI cluster configuration example"

# Avoid:
git commit -m "Update stuff"
git commit -m "Fix things"
git commit -m "docs" (too vague)
```

## Versioning

This project follows semantic versioning for releases:
- `MAJOR.MINOR.PATCH` (e.g., `1.0.0`)
- Increment MAJOR for incompatible changes
- Increment MINOR for new features
- Increment PATCH for bug fixes

When contributing:
- Reference the next expected version in discussions
- Note breaking changes clearly in PR description

## Questions or Need Help?

- **Documentation Questions:** Open a discussion or issue
- **Environment Issues:** Check troubleshooting docs first
- **General Questions:** Use GitHub Discussions (when available)
- **Security Issues:** See below

## Security

If you discover a security vulnerability:

1. **Do not** open a public issue
2. Email details to the maintainers privately
3. Include steps to reproduce and potential impact
4. Allow time for a fix before public disclosure

## Additional Resources

- [Nutanix Developer Community](https://www.nutanixdev.com/)
- [Harbor Documentation](https://goharbor.io/)
- [NKP Documentation](https://docs.d2iq.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

## License

By contributing to this project, you agree that your contributions will be licensed under the MIT License.

---

Thank you for making this project better! 🚀
