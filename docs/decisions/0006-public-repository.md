# ADR-0006: Keep the Project Repository Public

- **Status:** Accepted
- **Date:** 2026-11-10

## Context
The repository contains generic project documentation and is intended to be useful as a public software project and portfolio item. The target GitLab service is private, but that does not require this project's own repository to be private.

## Decision
Keep `majiddarvishan/gitlab-mcp` public.

## Publication rules
Never publish:
- Internal GitLab URLs, hostnames, IP addresses, DNS names, or network topology.
- Company project names, private source code, customer data, or operational details.
- Access tokens, credentials, private keys, cookies, or unredacted logs.
- Any information the user or organization has not authorized for public release.

Generic architecture, requirements, ADRs, examples using placeholders, and reusable implementation code may be public.

## Consequences
- Documentation must use neutral placeholders for private infrastructure.
- Review content for accidental disclosure before every commit.
- Public visibility is intentional, not an indication that the target GitLab is public.
