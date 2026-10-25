---
name: design-system-on-canvas
description: Build a design system in a Palet document, with variables in light and dark modes, components (buttons, inputs, cards) and their variants. Use when the user asks for a design system, a component library, a style guide or themed tokens in Palet.
---

If a Palet tool fails with an authentication error, tell the user to run `/mcp`, choose **palet** and sign in (no account: https://app.palet.so/sign-up). Don't ask for an API key.

## 1. Variables first

`set_variables(documentId, collection, modes, variables, id, baseVersion)`:

- One theme collection with `modes: ["light", "dark"]` for colours: `color/background`, `color/foreground`, `color/muted-foreground`, `color/primary`, `color/border`. Give every mode a value.
- A second collection, single mode, for numbers: `space/1…8`, `radius/sm|md|lg`, `text/sm|md|lg`. A string for the font (`font/body`).
- These names are the Tailwind theme of `write_html` (`bg-primary`, `gap-3`, `rounded-md`, `text-sm`, `font-body`), so name them for the classes you want to write.
- Use aliases for semantic names: `"color/button-bg": {"type": "color", "light": {"alias": "color/primary"}}`.
- If the repo already has tokens, take them from it (see the code-to-design skill) instead of inventing a palette.
- Then `get_variables`: you need the collection and mode ids for rendering.

## 2. Components on their own page

- Define each base component in `write_html` as a `<template data-component="Button">`; it is drawn on the Components page. Style it with theme classes only: `<template data-component="Button" class="flex items-center gap-2 px-4 h-10 rounded-md bg-primary"><p data-prop="label" class="text-sm font-medium text-background">Button</p></template>`.
- Name the layers you'll override (`data-prop="label"`, `data-prop="icon"`): they are the component's props. Text layers take text, other layers `true`/`false`, a nested instance swaps by component name, an icon (`<svg data-prop="icon" data-icon="plus">`) swaps by icon name.
- The result's `components` lists each `{id, name, props}`. For a frame drawn earlier, `create_component(documentId, nodeId, id, baseVersion)` makes it a component and returns its `props`.

## 3. Variants and examples

- Variants (Primary, Secondary, Destructive) are instances with props, not copies of the component: `<Button label="Save" icon="check"/>`. An instance's `class` sets only its box (`w-full`), as values.
- For a row of instances or a grid of states, place them by name in one write, or place one and `duplicate_nodes(nodeId, count, x, overrides)`.
- A specimen page (swatches, type scale, spacing) helps people use the system; draw it with the same theme classes.

## 4. Check both modes

- `render` the Components page. Then `render` it with `modes: "<theme collection id>:dark"`: same layers, dark values.
- `inspect` and fix any `unresolved_variable` (a binding with no value in that mode), `missing_component`, `text_overflow` or `clipped`. Read the write's `ignored` for classes or props that did nothing.
- To show dark next to light, `duplicate_nodes` a screen and set `props.modes` on the copy (`{"<collection id>": "dark"}`) with `apply_transaction`, or `data-modes` in HTML.

## Common mistakes

- **Literal values in components.** Anything left unbound won't change between modes. No inline `style`, no `var(--x, #fallback)`.
- **The collection name used as an id.** `modes` takes `vc_…:<mode id>`, from `get_variables`.
- **Children in an instance.** `<Button/>` holds nothing. Layers can't be added to or removed from an instance; change the component or `detach_instance`.
- **A variant drawn as a separate frame.** It stops following the component. Use an instance with props.
- **A name used twice in one write, or an unknown one.** It is refused with the names that exist.
- **Too many ops in one call.** More than 1,000 operations is refused with `too_large`: one component per call, and very large `set_variables` in batches.
