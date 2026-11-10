# ADR-0004: Implementation Language

- **Status:** Proposed
- **Date:** 2026-11-10

## Context
The project needs a maintainable local MCP server with good HTTP client support and straightforward distribution as a single executable.

## Options
- Go
- TypeScript/Node.js
- Python

## Proposal
Use Go for the MCP server.

## Rationale
- A single compiled binary is convenient for local desktop integration.
- Go has mature HTTP, TLS, concurrency, and JSON support.
- The user has substantial Go development experience.

## Consequences if accepted
- Go becomes the primary implementation language.
- The selected MCP Go SDK and its maturity, protocol coverage, and maintenance status must be evaluated before implementation.

This decision remains Proposed until explicitly approved.
