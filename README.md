# woku AI Toolkit

Connect your AI agent to [woku](https://woku.app), the voice-of-customer platform.

The toolkit gives your agent woku's full tool catalog through woku's MCP server, plus the woku method so it acts as a customer-experience consultant: design a listening program for your customer journey, place the fast VoC instruments (NPS, CSAT, CES, wokus) at each moment of truth, and turn every signal into a support ticket or an action plan.

It bundles:

- The **woku MCP server** connection (`https://api.woku.app/mcp`), so the agent gets the full catalog (143 tools) with woku's OAuth. No API keys to copy.
- The **woku skill**, which primes the agent with the method and tells it to call `woku_guide` first.

## Install

### Claude Code

Add woku's marketplace, then install the plugin. This ships both the MCP server and the skill:

```
/plugin marketplace add wokuApp/woku-ai-toolkit
/plugin install woku@woku-ai-toolkit
```

Once the toolkit is accepted into the official marketplace, `/plugin install woku@claude-plugins-official` also works.

### OpenAI Codex

Add the woku MCP server to `~/.codex/config.toml` and authenticate:

```toml
[mcp_servers.woku]
url = "https://api.woku.app/mcp"
```

```
codex mcp login woku
```

`auth = "oauth"` is the default for a bare `url`, so Codex opens the browser to complete woku's OAuth. Check it with `codex mcp list`.

### OpenCode

Add the woku MCP server to your `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "woku": {
      "type": "remote",
      "url": "https://api.woku.app/mcp",
      "enabled": true
    }
  }
}
```

OpenCode runs OAuth automatically on the first authenticated call. To start the login yourself, run `opencode mcp auth woku`.

### OpenClaw

Install the plugin (this ships the woku skill):

```
openclaw plugins install npm:@wokuapp/ai-toolkit
```

You can also install from the git repository with `openclaw plugins install git:github.com/wokuApp/woku-ai-toolkit`.

OpenClaw's plugin manifest declares local MCP servers only, so add woku's remote MCP server (`https://api.woku.app/mcp`) through your OpenClaw MCP configuration.

### Pi

The package exposes the woku skill through Pi's `pi` package convention. Install `@wokuapp/ai-toolkit` from npm in your project and Pi loads the skill. Add woku's MCP server (`https://api.woku.app/mcp`) through Pi's MCP configuration.

### claude.ai and ChatGPT

These are configured from the app, not from a package. Add a custom connector (claude.ai) or a developer-mode connector (ChatGPT) with the URL `https://api.woku.app/mcp`.

### Any other MCP client

Point it at `https://api.woku.app/mcp` and approve access from your woku account.

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
