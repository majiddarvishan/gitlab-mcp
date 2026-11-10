# ADR-0007: ChatGPT Desktop First, Codex as Supported Fallback

- **Status:** Accepted
- **Date:** 2026-11-10

## Context
The user's preferred workflow is ChatGPT Desktop in Chat mode. The user is willing to switch to Codex if required for local MCP compatibility or capabilities.

## Decision
Prioritize a workflow that works in ChatGPT Desktop. Codex is an acceptable alternative client and should be supported where the host's local MCP configuration permits it.

Do not assume that ChatGPT web custom MCP apps and local STDIO MCP integrations are the same feature. Validate the exact current support matrix for ChatGPT Desktop Chat mode and Codex before finalizing client setup.

## Consequences
- Client compatibility is a first-class requirement.
- The server should avoid unnecessary client-specific behavior.
- The documentation must clearly state any client-specific limitations and fallback steps.
