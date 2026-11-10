# ADR-0002: Local MCP Architecture

- **Status:** Accepted
- **Date:** 2026-11-10

## Context
The workstation running ChatGPT Desktop can directly access the internal GitLab instance. A remote relay or public tunnel is therefore not required for the initial use case.

## Options
1. Local MCP process connecting directly to GitLab.
2. Internal MCP service exposed through a secure outbound tunnel.
3. Publicly expose GitLab or a proxy.

## Decision
Use a local MCP process on the user's workstation. It will call the self-managed GitLab API over the workstation's existing internal network or VPN connection.

Do not expose GitLab to the public internet as part of this design.

## Consequences
- The local process depends on the workstation being online and able to reach GitLab.
- GitLab credentials are held locally and must be protected.
- A tunnel or always-on internal service can be considered later only if a new requirement justifies it.
- The exact local MCP client integration and configuration format must be verified against current OpenAI documentation before implementation.
