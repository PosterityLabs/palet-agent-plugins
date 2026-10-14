---
description: What to do when Palet's MCP tools cannot connect or are not signed in
alwaysApply: false
---

Palet's tools come from its hosted MCP server, `https://api.palet.so/mcp`. People sign in with their Palet account through the browser; there is no API key.

- If a Palet tool fails with an authentication error (401, "unauthorized", "needs authentication"), or the Palet tools are missing, tell the user to run `/mcp`, choose **palet** and sign in. In Codex, they sign in when the plugin is installed, or with `codex mcp login palet`.
- If they don't have a Palet account yet, point them to https://app.palet.so/sign-up.
- The agent works in the one workspace the user picked when they approved it. If a document they mention is missing from `list_documents`, it may be in another workspace: they can sign in again and choose that one.
- Don't ask for an API key, and don't edit MCP config files to add one.
