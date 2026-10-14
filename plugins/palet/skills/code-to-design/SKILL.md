---
name: code-to-design
description: Build a design in Palet from this codebase, using its real tokens (colours, spacing, type, radii) as Palet variables. Use when the user asks to design a screen, page or component in Palet "from my code", "with my tokens" or "in our style".
---

If a Palet tool fails with an authentication error, tell the user to run `/mcp`, choose **palet** and sign in (no account: https://app.palet.so/sign-up). Don't ask for an API key.

## 1. Read the codebase first

Find the design language before touching Palet: CSS custom properties, a Tailwind config or `@theme` block, a tokens file (JSON, TS), a theme object. Note colours, spacing scale, radii, font families and sizes, and whether there's a dark theme. Look at one or two existing components for layout habits (card padding, button height). Don't invent values the repo doesn't have.

## 2. Pick or make the document

- `whoami` once: it says which workspace you're in and what you may do.
- `list_documents`, and reuse a document the user names. Otherwise `create_document(name, requestId)`. It returns `id`, `pageId` and `version`; the `requestId` makes a retry return the same document instead of a second one.

## 3. Tokens become variables, in one call

`set_variables(documentId, collection, modes, variables, id, baseVersion)`:

- One collection per token family is fine ("Color", "Space"), or one "Tokens" collection. If the repo has a dark theme, pass `modes: ["light", "dark"]` and give both values; otherwise one mode.
- Names use `/` for groups: `color/accent`, `space/4`, `radius/md`. Keep the repo's own names so design-to-code maps back cleanly.
- Then `get_variables` and use each variable's `cssName`. Don't derive it yourself: a clash gets a suffix.

## 4. Draw with write_html

`write_html(documentId, html, parentId: <pageId>, baseVersion, id, check: true)`:

- Bind every token value as `var(--color-accent, #5856d6)`: the name from `cssName`, and the literal fallback the layer keeps.
- Root elements go on the page with `left` / `top`. Give each a `data-name` ("Settings / Desktop").
- Draw one screen per call. A large screen can go in two calls: the frame first, then its sections with `parentId` set to that frame's id from `created`.
- Icons: `search_icons`, then `insert_icon` into the frame, rather than hand-written SVG.

## 5. Check before you say it's done

- The write's result has `issues` (from `check: true`). Fix every `text_overflow`, `clipped`, `font_missing` and `unresolved_variable` before going on.
- `render` the screen and look at it. Compare it with the codebase's real UI (spacing, weights, alignment), not with what you meant to draw.
- With modes, `render` again with `modes: "<collection id>:dark"` (the collection id from `get_variables`).
- Tell the user the document name and what you built. The document is live: they see it in Palet as you write.

## Common mistakes

- **Bare text in a `<div>`** is refused. Wrap it in `<p>`.
- **`flex: 1`** stays raw CSS. Write `flex-grow: 1; min-width: 0`.
- **`left` / `top` on a nested layer** do nothing unless it has `position: absolute`.
- **Frames clip their children.** A shadow or a badge that sticks out needs `overflow: visible` on the parent.
- **Stale reads.** Each result carries a `version`: use it as the next `baseVersion`. On `stale_base`, read again. On `pending`, call `get_transaction`, then send the same call with the same `id`.
- **Hard-coded values** where the repo has a token. Every colour, gap, radius and font size should be a `var(...)`.
