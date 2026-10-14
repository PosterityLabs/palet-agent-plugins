---
name: design-system-on-canvas
description: Build a design system in a Palet document, with variables in light and dark modes, components (buttons, inputs, cards) and their variants. Use when the user asks for a design system, a component library, a style guide or themed tokens in Palet.
---

If a Palet tool fails with an authentication error, tell the user to run `/mcp`, choose **palet** and sign in (no account: https://app.palet.so/sign-up). Don't ask for an API key.

## 1. Variables first

`set_variables(documentId, collection, modes, variables, id, baseVersion)`:

- One theme collection with `modes: ["light", "dark"]` for colours: `color/bg`, `color/fg`, `color/muted`, `color/accent`, `color/border`. Give every mode a value.
- A second collection, single mode, for numbers: `space/1…8`, `radius/sm|md|lg`, `font/size/sm|md|lg`. Strings for font families (`font/family/body`).
- Use aliases for semantic names: `"color/button/bg": {"type": "color", "light": {"alias": "color/accent"}}`.
- If the repo already has tokens, take them from it (see the code-to-design skill) instead of inventing a palette.
- Then `get_variables`: you need each `cssName`, and the collection and mode ids for rendering.

## 2. Components on their own page

- Draw each base component with `write_html` and `check: true`, on a page named "Components". Bind every value: `background: var(--color-button-bg, #5856d6); padding: var(--space-3, 12px) var(--space-4, 16px)`.
- Name layers you'll override (`data-name="Label"`, `data-name="Icon"`).
- `create_component(documentId, nodeId, id, baseVersion)` on each frame. Keep the `layers` it returns: their ids are the override keys.

## 3. Variants and examples

- Variants (Primary, Secondary, Destructive) are instances with overrides, not copies of the component. In HTML: `<div data-instance="fr_button" style="width: fit-content"></div>`, then `get_html` it, change the `data-ref` layer's text or style, and write it back.
- For a row of instances or a grid of states, place one and `duplicate_nodes(nodeId, count, x, overrides)`; `overrides` maps the original's layer ids to patches.
- A specimen page (swatches, type scale, spacing) helps people use the system; draw it with the same variables.

## 4. Check both modes

- `render` the Components page. Then `render` it with `modes: "<theme collection id>:dark"`: same layers, dark values.
- `inspect` and fix any `unresolved_variable` (a binding with no value in that mode), `missing_component`, `text_overflow` or `clipped`.
- To show dark next to light, `duplicate_nodes` a screen and set `props.modes` on the copy (`{"<collection id>": "dark"}`) with `apply_transaction`.

## Common mistakes

- **Literal values in components.** Anything left unbound won't change between modes.
- **The collection name used as an id.** `modes` takes `vc_…:<mode id>`, from `get_variables`.
- **Children in an instance.** Write `<div data-instance>` with no children. Layers can't be added to or removed from an instance; change the component or `detach_instance`.
- **A variant drawn as a separate frame.** It stops following the component. Use an instance with overrides.
- **Too many ops in one call.** More than 1,000 operations is refused with `too_large`: one component per call, and very large `set_variables` in batches.
