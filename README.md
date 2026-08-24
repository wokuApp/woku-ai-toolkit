# woku AI Toolkit

Connect your AI agent to [woku](https://woku.app), the voice-of-customer platform.

The toolkit gives your agent woku's full tool catalog through woku's MCP server, plus the woku method so it acts as a customer-experience consultant: design a listening program for your customer journey, place the fast VoC instruments (NPS, CSAT, CES, wokus) at each moment of truth, and turn every signal into a support ticket or an action plan.

It bundles:

- The **woku MCP server** connection (`https://api.woku.app/mcp`), so installing the toolkit exposes the full catalog (143 tools) with woku's OAuth. No API keys to copy.
- The **woku skill**, which primes the agent with the method and tells it to call `woku_guide` first.

## Install

- **Claude Code**: add woku's marketplace, then install the plugin:

  ```
  /plugin marketplace add wokuApp/woku-ai-toolkit
  /plugin install woku@woku-ai-toolkit
  ```

  Once the toolkit is listed in the official marketplace, `claude plugin install woku@claude-plugins-official` also works.

- **OpenAI Codex**: add the woku MCP server to `~/.codex/config.toml`:

  ```toml
  [mcp_servers.woku]
  url = "https://api.woku.app/mcp"
  ```

  Codex handles woku's OAuth automatically (`auth = "oauth"` by default). Once the toolkit is listed in a Codex marketplace, `codex plugin add woku` will install the packaged skill as well.

- **OpenCode**: install the package from the git mirror:

  ```
  openclaw plugins install git:github.com/wokuApp/woku-ai-toolkit
  ```

  Once published to npm, `openclaw plugins install npm:@woku/ai-toolkit` also works. Alternatively, add the server directly in `opencode.json`:

  ```json
  {
    "mcp": {
      "woku": {
        "type": "remote",
        "url": "https://api.woku.app/mcp",
        "enabled": true
      }
    }
  }
  ```

- **claude.ai and ChatGPT**: these are configured from the app, not from a package. Add a custom connector (claude.ai) or a developer-mode connector (ChatGPT) with the URL `https://api.woku.app/mcp`.

- **Any other MCP client**: point it at `https://api.woku.app/mcp` and approve access from your woku account.

See the [connect guide](https://docs.woku.app/mcp/connect-a-client) and the [agent guide](https://docs.woku.app/mcp/agent-guide) for details.

## What you get

Once connected, ask your agent things like:

- "I run an ecommerce, how do I start listening to my customers with woku?"
- "Design a VoC program for my customer journey."
- "How is my NPS this month, and should I open an action plan?"
- "Summarize last week's reviews and the top improvement themes."

Every connection is bound to one company, chosen when you approve access. Read tools need no special permission; write tools require the `mcp:write` scope, and destructive tools or sends require an explicit confirmation.

## License

MIT. See [LICENSE](./LICENSE).
