# Security Policy

## Reporting Vulnerabilities

If you discover a security vulnerability in this repository, please **do not** open a public GitHub issue.

Instead, please email us at security@elektryk-opole.org with:
- Description of the vulnerability
- Steps to reproduce (if applicable)
- Potential impact
- Suggested fix (if you have one)

We will acknowledge your report within 48 hours and work toward a fix.

## Security Features

This repository has the following security features enabled:

- ✅ Dependabot vulnerability scanning (weekly checks for dependencies)
- ✅ GitHub Advanced Security (secret scanning, code scanning)
- ✅ Branch protection on `main` (requires reviews before merge)
- ✅ Enforcement of signed commits (recommended)
- ✅ CODEOWNERS enforcement

## Guidelines

- Keep dependencies up-to-date
- Review and merge Dependabot PRs promptly
- Never commit secrets (API keys, tokens, passwords)
- Use environment variables for sensitive configuration
- Enable two-factor authentication (2FA) on your GitHub account

## Additional Resources

- [GitHub Security Documentation](https://docs.github.com/en/code-security)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CIS Kubernetes Benchmarks](https://www.cisecurity.org/benchmark/kubernetes)
