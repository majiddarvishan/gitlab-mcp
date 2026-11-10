# ADR-0001: Project Goals and Boundaries

- **Status:** Accepted
- **Date:** 2026-11-10

## Context
The user wants to use ChatGPT Desktop to work with a self-managed GitLab instance available only on the company's internal network. The workstation running ChatGPT Desktop can already reach that GitLab instance.

## Decision
Build a local MCP integration for GitLab. The project should support AI-assisted software development and GitLab project workflows while keeping the GitLab service itself private.

The public source repository is a generic project repository. It must not contain company-specific internal details or secrets.

## Consequences
- No public exposure of the company's GitLab is required.
- The local workstation is the network bridge to GitLab.
- Requirements and architecture are documented before implementation.
- The design must account for the fact that repository content supplied to a cloud-hosted model may be transmitted to the model provider for processing.
