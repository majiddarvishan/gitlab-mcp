# ADR-0012: Read-Only CI/CD; Defer Job Logs and Artifacts

- **Status:** Accepted
- **Date:** 2026-11-10
- **Decision owner:** User

## Context
CI/CD provides useful visibility into development status, but pipeline execution and job logs/artifacts may have operational side effects or contain credentials, deployment information, and other sensitive data.

## Options
1. Include CI/CD reads and mutations, logs, and artifacts from the first release.
2. Include only pipeline/job metadata and status; defer execution controls, logs, and artifacts.

## Decision
The first release is read-only for CI/CD.

Allowed:
- Read pipeline metadata and status.
- Read job metadata and status.

Excluded from the first release:
- Triggering pipelines.
- Retrying or cancelling pipelines/jobs.
- Reading job logs/traces.
- Downloading job artifacts.
- Reading or changing CI/CD variable values, including secrets.

Job logs and artifacts are deferred until a separate review defines data classification, redaction, response-size limits, and handling requirements.

## Consequences
- CI/CD write tools are not part of the initial tool surface.
- Job trace and artifact endpoints are not called in the first release.
- Pipeline/job responses must be limited to metadata/status fields required for workflow visibility.
- Future expansion requires a new decision and security review.

## Reference
- [GitLab Pipelines API](https://docs.gitlab.com/api/pipelines/)
