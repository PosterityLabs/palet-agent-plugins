---
name: design-to-code
description: Implement a Palet design in this codebase, using the repo's own components, tokens and conventions. Use when the user asks to build, code or implement a screen or component "from Palet", or links a Palet document.
---

If a Palet tool fails with an authentication error, tell the user to run `/mcp`, choose **palet** and sign in (no account: https://app.palet.so/sign-up). Don't ask for an API key.

## 1. Find the design

- `list_documents`, and match the name the user gave. A link like `app.palet.so/doc/<id>` has the document id in it.
- `get_tree(documentId)` for the outline (pages, frames, names). Go deeper with `get_tree(documentId, nodeId)` where `childCount` shows more.
- `find_nodes(documentId, name: "...")` to jump to a named frame. If several frames match ("Login / A", "Login / B"), ask which one.

## 2. Read it three ways

- `get_html(documentId, nodeId)`: the structure and every style, inline. It's flexbox and grid, close to what you'll write.
- `get_variables(documentId)`: what each `var(--name, fallback)` in that HTML means, per mode.
- `render(documentId, nodeId)`: what it looks like. Use it to settle anything the HTML leaves unclear.

## 3. Map it to the repo, don't transcribe it

- **Variables become tokens.** For each `var(--color-accent, #5856d6)`, find the repo's token with that name or value. Use the token; never paste the fallback literal. If the repo has no match, say so and propose adding one.
- **Components become components.** A `<div data-instance="fr_…">` is a placed component. Look for the repo's existing component (Button, Card, Tab bar) and pass the instance's overrides as props. Layers inside it have `data-ref`. Only write a new component when nothing fits.
- **Layout carries over.** Keep `display: flex` / `grid`, gap and padding as the repo expresses them (Tailwind classes, CSS modules, styled props). Drop canvas-only details: `left` / `top` of a layer of the page, `data-id`, `data-name`.
- **Icons:** a vector named `Icon / <name>` is a Lucide icon. Use the repo's icon library with that name.
- **Text:** take copy from the HTML exactly; don't shorten it.

## 4. Check before you say it's done

- Run the repo's type check and tests.
- If the app can run, look at the result next to `render`'s image: spacing, sizes, weights, colours in each mode.
- List what you couldn't map (a missing token, a component the repo lacks) instead of approximating silently.

## Common mistakes

- **`get_document` for reading a design.** It returns raw nodes and is large; `get_html` with `nodeId` is the readable form.
- **Literal colours and sizes** copied from fallbacks, when the repo has tokens.
- **Rebuilding an instance's insides** instead of using the component it points to.
- **Pixel positions.** `left` / `top` on a nested layer exist only with `position: absolute`; everything else is flow layout. Don't add absolute positioning the design doesn't have.
