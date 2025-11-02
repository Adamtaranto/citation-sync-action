# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in Citation Sync Action, please report it privately to help protect the community.

### How to Report

**DO NOT** open a public issue for security vulnerabilities.

Instead, please report security issues by:

1. **GitHub Security Advisories** (Preferred):
   - Go to the [Security tab](https://github.com/Adamtaranto/citation-sync-action/security)
   - Click "Report a vulnerability"
   - Fill out the form with details

2. **Direct Contact**:
   - Contact @Adamtaranto directly via GitHub
   - Include "SECURITY" in the subject line
   - Provide detailed information about the vulnerability

### What to Include

When reporting a vulnerability, please include:

- **Description** of the vulnerability
- **Steps to reproduce** the issue
- **Potential impact** of the vulnerability
- **Affected versions** (if known)
- **Suggested fix** (if you have one)
- **Your contact information** for follow-up

## Response Timeline

- **Initial Response**: Within 48 hours
- **Status Update**: Within 7 days
- **Fix Timeline**: Depends on severity
  - Critical: Within 7 days
  - High: Within 14 days
  - Medium: Within 30 days
  - Low: Next regular release

## Disclosure Policy

- Security vulnerabilities will be handled privately until a fix is available
- We will coordinate with you on the disclosure timeline
- After a fix is released, we will publish a security advisory
- Credit will be given to reporters (unless anonymity is requested)

## Supported Versions

We support the following versions with security updates:

| Version | Supported          |
| ------- | ------------------ |
| 1.x.x   | :white_check_mark: |
| < 1.0   | :x:                |

We recommend always using the latest stable release.

## Security Best Practices for Users

When using this action:

1. **Use Specific Versions**: Pin to specific versions or major version tags (e.g., `v1`)
2. **Review Permissions**: Ensure `contents: write` permission is necessary for your use case
3. **Protect Tokens**: Use `secrets.GITHUB_TOKEN` or carefully manage PATs
4. **Enable Branch Protection**: Protect your default branch to prevent unauthorized changes
5. **Review Changes**: Review the action's code before using it in production
6. **Monitor Updates**: Watch for security advisories and update regularly

## Known Security Considerations

### Token Permissions

This action requires `contents: write` permission to:
- Commit changes to CITATION.cff
- Create and push tags
- Push to the default branch

**Mitigation**: The action only modifies CITATION.cff and related tags. It does not access other files or secrets.

### Tag Creation

This action creates new tags automatically.

**Mitigation**: Tag creation only occurs when CITATION.cff needs updating. The action validates all changes before committing.

### Workflow Loops

The action could potentially trigger itself in a loop.

**Mitigation**: Commits include `[skip ci]` by default to prevent workflow re-triggering. The workflow also checks if the last commit was made by the action.

## Dependencies

This action has minimal dependencies:
- GitHub Actions checkout action
- Standard bash/git utilities available in GitHub Actions runners

We monitor dependencies for security vulnerabilities and update them as needed.

## Acknowledgments

We appreciate security researchers and users who help keep Citation Sync Action secure. Thank you for reporting vulnerabilities responsibly!
