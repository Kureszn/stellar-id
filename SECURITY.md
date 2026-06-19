# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in StellarID, please do **not** open a public GitHub issue.

Email the maintainer with:
- A description of the vulnerability
- Steps to reproduce
- Potential impact and severity

We will respond within 48 hours and coordinate a fix before any public disclosure.

## Supported Versions

| Version | Supported |
|---|---|
| 0.1.x | ✅ Yes |

## Known Limitations

This contract is currently pre-audit. Do not use in production with real credentials or financial gating until a full security audit is completed.

## Scope

In-scope for security reports:
- `contracts/stellar-id/src/lib.rs` — all contract functions
- Unauthorized credential issuance or revocation
- Credential forgery or replay attacks
- Auth bypass on admin or issuer functions
- Storage manipulation or data corruption

Out of scope:
- Documentation errors
- Issues in third-party Soroban SDK dependencies
- Theoretical attacks with no practical exploit path

## Severity Guidelines

| Severity | Example |
|---|---|
| Critical | Auth bypass allowing anyone to issue credentials |
| High | Issuer can revoke another issuer's credentials |
| Medium | Reputation score manipulation |
| Low | Missing event emission |
