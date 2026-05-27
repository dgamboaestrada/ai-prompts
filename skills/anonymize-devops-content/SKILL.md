---
name: anonymize-devops-content
description: Invoke whenever a user has existing content and wants internal or sensitive details removed before sharing it externally. This is the "clean this before I post it" skill. Trigger on any combination of: (1) user presents a file, script, config, runbook, or doc, and (2) wants to strip, remove, scrub, redact, or check for internal details like hostnames, credentials, account IDs, employee names, internal URLs, or org-specific references, before posting, pushing, publishing, or handing off. Also trigger for "is this safe to share?" questions about existing content. Works on any format: shell scripts, Terraform, Ansible, Dockerfiles, Kubernetes manifests, CI/CD configs, runbooks, postmortems. Do NOT trigger for: security code reviews, questions about how to store or manage secrets, encrypting files, auditing access controls, or generating new content.
license: MIT
metadata:
  author: Daniel Gamboa Estrada
  version: "0.1.0"
  category: security
  tags:
    - devops
    - infrastructure
    - scripts
    - documentation
    - security
    - anonymization
    - compliance
---

# DevOps Content Anonymization

When this skill triggers, take the provided DevOps content and return a fully anonymized version — one that preserves technical accuracy and usefulness while removing anything that could expose credentials, infrastructure topology, or organizational information.

Return the anonymized content directly (inline or as a file, matching how it was provided). Include a brief summary of what was changed.

## Quick Reference

| Category | Replace with |
|----------|--------------|
| Company domains | `example.com`, `api.example.com` |
| AWS Account IDs | `111111111111` |
| Certificate UUIDs | `12345678-1234-1234-1234-123456789012` |
| Serial numbers | `01BBBBBBBBBBBBBBBBBBBBBBBBBBBB` |
| ELB / resource names | `elb-xxxxx`, `resource-xxxxx` |
| Internal ticket IDs | `XXXX-XXXX` or remove context |
| Organization / employee names | Generic roles or omit |
| File paths | Standard system paths (`/opt/`, `/home/user/`) |
| IP addresses | `10.0.0.x` or `192.168.0.x` |
| Internal URLs | Generic formats or omit |

## What to Look For

**Infrastructure:**
- Domain names, hostnames, ELB / VPC / EC2 / RDS names
- AWS Account IDs in ARNs and API calls
- Certificate identifiers and serial numbers
- Internal IP addresses and CIDR blocks
- Terraform state identifiers and resource references

**Credentials:**
- AWS ARNs with account IDs
- SSH key names, API keys, tokens
- Database credentials and connection strings

**Organizational:**
- Company / department names, employee names, email addresses
- Internal ticket / issue IDs, project names
- Internal tool URLs and endpoints
- Specific dates revealing deployment timelines or incidents

## How to Anonymize

Work through the content systematically by type:

**Domains**: Replace with `example.com` subdomain structure. Use descriptive but generic subdomains (`api.example.com`, `db.example.com`). No environment prefixes.

**AWS Account IDs**: Always replace with `111111111111`. Keep surrounding ARN structure intact.

**Resource names**: Remove environment prefixes and infrastructure identifiers. Use `resource-xxxxx` or descriptive-but-generic names.

**Credentials / secrets**: Remove completely. Use `YOUR_API_KEY` or `YOUR_SECRET` if a placeholder is needed for clarity.

**File paths**: Use standard paths — `/opt/scripts/`, `/home/user/.ssh/id_rsa`, `/opt/app/config/`.

**Dates**: Use generic ISO dates. Remove timezone details or context that reveals incident timelines.

**Org context**: Replace company names with generic role descriptions. Remove ticket IDs entirely or replace with `XXXX-XXXX`.

## Common Patterns

**AWS ARN:**
```
# Before
arn:aws:acm:us-east-1:123456789012:certificate/abc12345-dead-beef-cafe-feedfacecafe

# After
arn:aws:acm:us-east-2:111111111111:certificate/12345678-1234-1234-1234-123456789012
```

**ELB / internal URL:**
```
# Before
prod-app-1234567890.us-east-1.elb.amazonaws.com
payments-svc.internal.companyname.com

# After
elb-xxxxx.us-east-2.elb.amazonaws.com
service.example.com
```

**File paths:**
```
# Before
/home/jsmith/deploy/prod-config.yml
/root/.ssh/company-key.pem

# After
/opt/deploy/config.yml
/home/user/.ssh/id_rsa
```

**Certificate serial number:**
```
# Before
3a:bb:f1:02:cc:44:d5:e6:f7:88:99:aa:bb:cc:dd:ee

# After
01:bb:bb:bb:bb:bb:bb:bb:bb:bb:bb:bb:bb:bb:bb:bb
```

## Principles

- **Be consistent**: use the same replacement for the same original value throughout
- **Preserve structure**: keep ARN formats, path structures, command syntax intact
- **Don't over-anonymize**: don't replace technical terms or generic concepts
- **Don't break examples**: anonymized code/commands should still be syntactically valid
- **No removal markers**: don't leave `[REDACTED]` or `# removed for security` comments — replace cleanly

## Final Check

Before returning, verify:
- No company / infrastructure-specific domains (only `example.com`)
- No real account IDs (only `111111111111`)
- No environment-specific resource names
- No employee, company, or organizational names
- No internal identifiers or tracking references
- No credentials, keys, or secrets
- Technical accuracy and functionality preserved
