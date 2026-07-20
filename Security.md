# Security Policy

## Supported Versions

Security fixes are applied to the latest version on the `main` branch only.

| Version | Supported |
|---------|-----------|
| Latest (main) | ✅ Yes |
| Older commits | ❌ No |

## Reporting a Vulnerability

**Do not open a public GitHub issue for security vulnerabilities.**

If you discover a security vulnerability in Nexora, please report it privately:

1. Go to the **Security** tab on the GitHub repository
2. Click **"Report a vulnerability"**
3. Fill in the details of the vulnerability

Alternatively, contact the maintainer directly through GitHub.

### What to include in your report

- A description of the vulnerability and its potential impact
- Steps to reproduce the issue
- Any proof-of-concept code if applicable
- Suggested fix if you have one

### What to expect

- Acknowledgement of your report within 48 hours
- An assessment of the severity and impact within 5 business days
- A fix or mitigation plan communicated privately before any public disclosure

We follow responsible disclosure — we will credit you in the fix commit and changelog unless you prefer to remain anonymous.

## Security Best Practices for Contributors

When contributing to Nexora, keep these in mind:

- Never commit real secrets, API keys, or credentials — use `.env` which is gitignored
- Always validate and sanitize user input with Zod before processing
- Use parameterized queries through Prisma and Mongoose — never build raw query strings from user input
- Refresh tokens must be rotated on every use — never reuse a refresh token
- Rate limiting is applied at the route level — do not remove it
- All JWT secrets must be cryptographically strong random strings (32+ characters)
