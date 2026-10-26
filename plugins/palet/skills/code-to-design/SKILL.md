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
- Names use `/` for groups and follow Palet's theme, so they become classes: `color/primary` (`bg-primary`), `space/4` (`gap-4`, `p-4`), `radius/md` (`rounded-md`), `text/sm` (`text-sm`), `font/body` (`font-body`). Keep the repo's own names inside those groups so design-to-code maps back cleanly.
- Then `get_variables` to see what exists and the collection and mode ids.

## 4. Draw with `write`

`write(documentId, markup, parentId: <pageId>, baseVersion: "latest", id, check: true)`:

- Style with Tailwind classes, so tokens bind by name: `<div class="flex gap-4 p-4 rounded-md bg-surface">`. A token without a theme class: `prefix-(--name)`, e.g. `shadow-(--shadow-card)`. No inline `style`, no literal colours, no `var(--x, #fallback)`.
- Root elements go on the page with `style="left: 80px; top: 80px"` (the one place inline style stays). Give each a `data-name` ("Settings / Desktop").
- A piece the repo has as a component (Button, Row, Card) is a `<template data-component="Row">` placed as `<Row title="General"/>`. A screen is templates plus instances: define each repeated piece once, then place it.
- Draw one screen per call; a large one can go in sections, each with `parentId` set to the frame's id from `created`, in parallel with `baseVersion: "latest"`.
- Icons: `<svg data-icon="search" class="size-4"></svg>` by name (`search_icons` to find one).

## 5. Check before you say it's done

- The write's result has `issues` (from `check: true`) and `ignored` (classes that did nothing). Fix every `text_overflow`, `clipped`, `font_missing` and `unresolved_variable` before going on.
- `render` the screen and look at it. Compare it with the codebase's real UI (spacing, weights, alignment), not with what you meant to draw.
- With modes, `render` again with `modes: "<collection id>:dark"` (the collection id from `get_variables`).
- Tell the user what you built. The document is live: they see it in Palet as you write.
- Then call `get_link` once, with the frames you drew and a one-line `summary`, and end your reply with its `card`, exactly as given.

## Common mistakes

- **Bare text in a `<div>`** is refused. Wrap it in `<p>`.
- **`left` / `top` on a nested layer** do nothing unless it has `absolute`.
- **Frames clip their children.** A shadow or a badge that sticks out needs `overflow-visible` on the parent.
- **Stale reads.** Each result carries a `version`: use it as the next `baseVersion`. On `stale_base`, read again. On `pending`, call `get_transaction`, then send the same call with the same `id`.
- **Hard-coded values** where the repo has a token. Every colour, gap, radius and font size should be a theme class or `prefix-(--name)`.
