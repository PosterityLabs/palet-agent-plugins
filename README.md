# Palet agent plugins

Use [Palet](https://palet.so) with your coding agent. The plugin connects your agent to Palet's hosted MCP server for every project, and adds skills for designing with it:

- **design-on-canvas**: draw anything (screens, mockups, flows) with every colour and size bound to variables
- **code-to-design**: build a design from your codebase's tokens and styles
- **design-to-code**: implement a Palet design with your repo's components and tokens
- **design-system-on-canvas**: variables in light and dark modes, components and variants
- **review-and-fix**: find and fix clipped text, overflow, broken bindings and hard-coded colours

Every skill draws with Palet variables, never hard-coded colours, so a theme, a dark mode or a fix is one edit. In Cursor, an always-on rule says the same.

You sign in with your Palet account in the browser the first time a tool runs. There is no API key. No account yet? [Sign up](https://app.palet.so/sign-up).

## Claude Code

```sh
claude plugin marketplace add palet-so/plugins
claude plugin install palet@palet
```

Then run `/mcp`, choose **palet** and sign in.

Without the plugin (the MCP server only, no skills), add it for every project with `--scope user`:

```sh
claude mcp add --scope user --transport http palet https://api.palet.so/mcp
```

## Codex

```sh
codex plugin marketplace add palet-so/plugins
codex plugin add palet@palet
```

You can also browse and install plugins with `/plugins` inside Codex after adding the marketplace.

Without the plugin (applies to every project):

```sh
codex mcp add palet --url https://api.palet.so/mcp
codex mcp login palet
```

## Cursor

Add this repository as a plugin marketplace, then install **palet**. See [installing Cursor plugins](https://cursor.com/docs/plugins#installing-plugins).

## Other agents

Add the server to your agent's user-level MCP config (not a project's), so it works in every folder:

```json
{
  "mcpServers": {
    "palet": {
      "type": "http",
      "url": "https://api.palet.so/mcp"
    }
  }
}
```

## Development

```sh
bun install
bun run build   # validates the Cursor schema and structure, and the Codex manifest
```

Try it locally before publishing:

```sh
claude plugin marketplace add ./
claude plugin install palet@palet
```
