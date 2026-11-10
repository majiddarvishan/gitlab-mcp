# ADR-0005: STDIO as Primary Transport

- **Status:** Proposed
- **Date:** 2026-11-10

## Context
The MCP server is intended to run locally and be launched by a compatible desktop/IDE host.

## Options
- STDIO
- Local HTTP
- Streamable HTTP on a network endpoint

## Proposal
Use STDIO as the primary transport for the local process integration.

## Rationale
- No listening network port is needed.
- The host can launch and supervise the process.
- It fits the local workstation architecture.

## Consequences if accepted
- Standard output must be reserved exclusively for MCP protocol messages.
- Logs and diagnostics must go to standard error.
- Client compatibility and configuration must be validated for ChatGPT Desktop and Codex before implementation.

This decision remains Proposed until explicitly approved.
