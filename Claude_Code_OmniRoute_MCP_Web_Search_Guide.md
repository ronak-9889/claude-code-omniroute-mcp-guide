claude-code-omniroute-mcp-guide

solution to bug diegosouzapw/OmniRoute#1882



# Claude Code + OmniRoute: Enabling Web Search Through MCP

*A practical troubleshooting guide for Windows + VS Code*

## Summary

I tested web search in Claude Code through a locally hosted OmniRoute MCP server at `http://localhost:20128`. Claude Code discovered 110 MCP tools and invoked `mcp__omniroute__omniroute_web_search` to retrieve search results.

The important distinction: configuring an MCP server is separate from configuring a model API endpoint, and a connected MCP server does not prove every tool works.

## The issue

- The MCP endpoint was `http://localhost:20128/api/mcp/stream`.
- Authentication initially failed with HTTP 401 and then HTTP 403 / `AUTH_001: Invalid management token`.
- The token supplied in the Authorization header was not accepted as a management token. A normal model API key should not be assumed to work for management/MCP authentication.
- After correcting the management-access/authentication configuration, Claude Code reported the server as connected and listed 110 tools.
- A real search test called `mcp__omniroute__omniroute_web_search` and returned results with URLs.

## Step-by-step

1. **Start OmniRoute.** Confirm the local service is available at `http://localhost:20128`.
2. **Use the MCP endpoint.** This setup used `http://localhost:20128/api/mcp/stream`, not the model API endpoint such as `/v1`.
3. **Fix authentication.** For `AUTH_001: Invalid management token`, check management/security settings, enable management access if required, and use the token expected by the MCP endpoint. Exact setting names may vary by version.
4. **Register the MCP server.** Example command shape (adapt to your version and auth method):
   ```powershell
   claude mcp add-json omniroute '{"type":"http","url":"http://localhost:20128/api/mcp/stream","headers":{"Authorization":"Bearer YOUR_MANAGEMENT_TOKEN"}}' -s local
   ```
   Replace the placeholder with the correct token. Never publish real credentials.
5. **Check `/mcp` in Claude Code.** In this test, OmniRoute showed as connected and 110 tools were listed.
6. **Call an actual search tool.** Ask Claude Code to search a current topic and report the exact tool name, dates, and source URLs. The tool called in this test was `mcp__omniroute__omniroute_web_search`.
7. **Verify the results.** Open the cited pages and check dates, figures, and claims. Tool invocation does not guarantee factual accuracy.
8. **Check OmniRoute logs.** Inspect request logs, MCP audit/tool-call logs, or provider logs for timestamp, backend, status, duration, and errors. Log labels vary by version.

## Reusable test prompt

> Use an available MCP tool to perform a live web search for a current topic. Return three results with headlines, publication dates, and source URLs. State the exact tool name used. If no search tool is available, say so clearly. Open at least one source and verify its claims.

## Separate Auto Mode warning

Claude Code also warned that the session was not eligible for updated Auto Mode classifier billing because it used the custom base URL `http://localhost:20128`; `/status` showed `Auto mode server: Disabled`. This is a separate gateway-compatibility/billing issue, not a web-search failure. See Anthropic's current documentation: https://code.claude.com/docs/en/auto-mode-classifier-billing

## Quick troubleshooting

| Symptom | Check |
|---|---|
| HTTP 401 | Check that the required authentication header is present and valid. |
| HTTP 403 / `AUTH_001` | Check management access and use the correct management token. |
| MCP connected but search fails | Inspect tools and explicitly call the web-search tool. |
| Search results look wrong | Open source pages and verify dates, numbers, and claims. |
| Auto Mode billing warning | Treat it separately and consult Anthropic's documentation. |

## Security checklist

- Never publish API keys, management tokens, or Authorization headers.
- Use placeholders in public examples.
- Record software versions and redact secrets from logs.
- Distinguish tool discovery, tool execution, and fact verification.

## What this test established

Claude Code connected to a local OmniRoute MCP endpoint, discovered 110 tools, and invoked `mcp__omniroute__omniroute_web_search`. This demonstrates tool access in this setup; gateway logs provide independent confirmation, and returned facts should still be verified.

## References

- Anthropic Auto Mode classifier billing: https://code.claude.com/docs/en/auto-mode-classifier-billing
- Local OmniRoute dashboard: http://localhost:20128
