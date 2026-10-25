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

- `get_html(documentId, nodeId)`: the structure in Palet's compact form: Tailwind classes, instances by name with props (`<Row title="General"/>`). It's flexbox and grid, close to what you'll write. `style="…"` appears only for values with no class (`left`, `top`).
- `get_variables(documentId)`: what each theme class or `(--name)` in that HTML means, per mode.
- `render(documentId, nodeId)`: what it looks like. Use it to settle anything the HTML leaves unclear.

## 3. Map it to the repo, don't transcribe it

- **Variables become tokens.** A class bound to a variable (`bg-primary`, `gap-3`, `shadow-(--shadow-card)`) names a token: find the repo's token with that name or value and use it; never paste a literal. If the repo has no match, say so and propose adding one.
- **Components become components.** An instance printed by name (`<Row title="General" icon="mic"/>`) is a placed component. Look for the repo's existing component (Button, Card, Tab bar) and pass its props. Only write a new component when nothing fits; the definition is the `<template data-component>` on the Components page.
- **Layout carries over.** Keep flex / grid, gap and padding as the repo expresses them (Tailwind classes carry over almost as they are; CSS modules or styled props otherwise). Drop canvas-only details: `left` / `top` of a layer of the page, `data-id`, `data-name`.
- **Icons:** `<svg data-icon="mic">` (or an `icon="mic"` prop) is a Lucide icon. Use the repo's icon library with that name.
- **Text:** take copy from the HTML exactly; don't shorten it.

## 4. Check before you say it's done

- Run the repo's type check and tests.
- If the app can run, look at the result next to `render`'s image: spacing, sizes, weights, colours in each mode.
- List what you couldn't map (a missing token, a component the repo lacks) instead of approximating silently.

## Common mistakes

- **`get_document` for reading a design.** It returns raw nodes and is large; `get_html` with `nodeId` is the readable form.
- **Literal colours and sizes** (`bg-[#…]`, `h-[52px]`) copied over, when the repo has tokens.
- **Rebuilding an instance's insides** instead of using the component it points to.
- **Pixel positions.** `left` / `top` on a nested layer exist only with `absolute`; everything else is flow layout. Don't add absolute positioning the design doesn't have.
