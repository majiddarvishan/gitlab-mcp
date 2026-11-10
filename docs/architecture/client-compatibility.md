# ChatGPT Desktop and Codex MCP Compatibility

- **Research status:** Official documentation reviewed
- **Reviewed on:** 2026-11-10
- **Runtime smoke test:** Pending on the target workstation
- **Primary surface:** New ChatGPT desktop app, Chat experience
- **Fallback surface:** Codex in the desktop app, Codex CLI, or supported IDE extension

## Executive finding

Current OpenAI documentation states that the ChatGPT desktop app, Codex CLI, and Codex IDE extension support MCP servers and share MCP configuration for the same Codex host.

The desktop app can add a local MCP server from **Settings > MCP servers**. The user can select STDIO or Streamable HTTP, provide the command or URL, save, and restart. The composer can show connected MCP servers with `/mcp`.

Therefore, one local MCP server can be configured once and reused by the supported ChatGPT desktop and Codex surfaces. Separate Chat and Codex server implementations are not required.

## Client compatibility matrix

| Client / surface | Local MCP server | STDIO | Configuration | Status |
|---|---|---|---|---|
| New ChatGPT desktop app — Chat | Documented | Supported | Shared Codex host MCP configuration | Officially documented; runtime test pending |
| New ChatGPT desktop app — Codex | Documented | Supported | Shared Codex host MCP configuration | Officially documented; runtime test pending |
| Codex CLI / TUI | Documented | Supported | `~/.codex/config.toml` on Linux/macOS; platform-equivalent user config on other OSes | Officially documented |
| Codex IDE extension | Documented | Supported | Shared Codex configuration; project-level config only for trusted projects | Officially documented |
| ChatGPT web — ordinary Chat | Does not read local Codex config | No local process launch through this path | Remote MCP/plugin route | Not a target for local STDIO |
| ChatGPT Work cloud execution | Local MCP may not be available in cloud execution | Depends on executor/environment | Cloud/workspace policies apply | Not a baseline target |

## Desktop setup documented by OpenAI

The current setup flow for the new ChatGPT desktop app is:

1. Open **Settings**.
2. Select **MCP servers**.
3. Select **Add server**.
4. Enter a server name.
5. Select **STDIO** or **Streamable HTTP**.
6. Provide the local command or server URL.
7. Save and restart.
8. Use `/mcp` in the composer to inspect connected servers.

The server list also indicates whether a server is enabled and whether it requires OAuth.

## Shared Codex configuration

OpenAI documents the default Codex configuration file as:

```text
~/.codex/config.toml
```

The ChatGPT desktop app, Codex CLI, and IDE extension share this configuration. Project-scoped configuration can also be used through `.codex/config.toml` for trusted projects.

For a local STDIO server, the relevant configuration fields include:
- `command` — required launcher command.
- `args` — optional command arguments.
- `env` — environment variables supplied to the process.
- `env_vars` — environment variable names allowed and forwarded.
- `cwd` — optional working directory.

Do not put the GitLab token directly in a checked-in configuration file. The exact secret retrieval design is tracked separately by SEC-001 and SEC-002.

## Host-side approval settings

The Codex MCP configuration supports:
- `default_tools_approval_mode` with values `auto`, `prompt`, `writes`, or `approve`.
- Per-tool `approval_mode` overrides.

The `writes` mode prompts for tools that are not marked read-only. This is a useful host-side safeguard for GitLab mutation tools.

However, host-side prompting is not sufficient as the only enforcement layer for this project. The server must retain its own approval lifecycle and reject a mutation unless the exact operation and parameters have a valid, explicit approval. A model-generated statement must never count as user approval.

## Linux desktop availability note

OpenAI release notes state that the new ChatGPT desktop app is available in public preview on:
- Ubuntu 24.04 LTS and Ubuntu 26.04 LTS.
- Debian 13.
- Fedora 43 and Fedora 44.

The Linux app release notes also state that controlling other desktop applications is not yet supported in the Linux preview. That limitation does not by itself rule out a local MCP process, but the workstation distribution and installed app build must be checked during TST-009.

If the target workstation runs a Linux distribution outside the documented preview list, Codex CLI or a supported IDE extension may be the practical local fallback.

## Important distinctions

- This finding applies to the **new ChatGPT desktop app** and the Codex host integration described in the current documentation. Do not assume the older ChatGPT Classic app has the same MCP support.
- It does not make a local STDIO server available to ChatGPT web.
- It does not mean the MCP server runs in the cloud. In the selected local architecture, the process runs on the workstation and uses the workstation's internal route to GitLab.
- OpenAI's documented configuration support is not a substitute for checking actual tool invocation and approval prompts on the target workstation.

## Verification still required

The following remains an implementation-independent environment smoke test and is tracked by TST-009:
- Confirm the exact ChatGPT desktop app distribution and version on the workstation.
- Confirm **Settings > MCP servers** is present.
- Register a harmless test STDIO MCP server.
- Confirm it is visible in Chat mode through `/mcp`.
- Confirm it is also available in Codex without a second server implementation.
- Confirm the host prompts for non-read-only tools when `writes` approval mode is configured.
- Confirm that the server-side approval gate remains mandatory even if host approval settings are changed.

## Official references

- [OpenAI — Model Context Protocol](https://learn.chatgpt.com/docs/extend/mcp)
- [OpenAI — Codex configuration reference](https://learn.chatgpt.com/docs/config-file/config-reference)
- [OpenAI — ChatGPT release notes, Linux desktop availability](https://help.openai.com/en/articles/6825453-chatgpt-release-notes)
- [OpenAI — Moving to the new ChatGPT desktop app](https://help.openai.com/en/articles/20001276-moving-to-the-new-chatgpt-desktop-app)
