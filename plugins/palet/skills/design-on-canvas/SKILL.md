---
name: design-on-canvas
description: Draw anything in Palet (screens, mockups, flows, explorations, slides, UI states) with every colour, size and font bound to Palet variables. Use whenever you create or change a design in Palet and no more specific Palet skill applies, including quick mockups and throwaway explorations.
---

If a Palet tool fails with an authentication error, tell the user to run `/mcp`, choose **palet** and sign in (no account: https://app.palet.so/sign-up). Don't ask for an API key.

**Always use variables.** Every colour, and every spacing, radius and font value you use more than once, is a Palet variable, bound with `var(...)`. This holds for a quick mockup too: designs that start as sketches become the reference, and a hard-coded colour drifts the moment two screens use it. With variables, a theme change, a dark version or a consistency fix is one edit instead of a redraw.

## 1. Variables before the first layer

- `get_variables(documentId)` first. If the document has a collection, **use it**: bind to its variables, and add only what is missing with `set_variables` (an existing name is updated, not duplicated).
- If it has none, make one with `set_variables(documentId, collection, modes, variables, id, baseVersion)` before drawing:
  - `collection: "Theme"`, `modes: ["light", "dark"]`, and a value for both modes, even if the user only asked for one look.
  - Name by role, not by colour: `color/bg`, `color/surface`, `color/border`, `color/text`, `color/text-muted`, `color/accent`, `color/danger`. A colour with a meaning gets its own name (`color/selection`, `color/agent/1`), so the meaning is visible in the inspector.
  - A second single-mode collection for numbers you repeat: `space/1…8`, `radius/sm|md|lg`, `font/size/sm|md|lg`. A string for the font family (`font/family/body`).
- If the codebase has tokens, take them from it (the code-to-design skill) instead of inventing a palette.
- Then `get_variables` again and write each variable's `cssName`. Don't derive it yourself: a clash gets a suffix.

## 2. Bind as you draw

- In `write_html`, every colour is `var(--color-text, #19191b)`: the variable's `cssName` and the literal fallback the layer keeps. Spacing and radii likewise: `gap: var(--space-3, 12px); border-radius: var(--radius-md, 8px)`.
- Inside a gradient or a shadow, name the variable without a fallback: `box-shadow: 0 8px 24px var(--color-shadow)`.
- A new colour you need mid-design is a new variable first (`set_variables`), then used. Never "just this once" as a literal.
- Pass `check: true`; fix any `unresolved_variable` before going on.

## 3. Light and dark are modes, not copies

- A dark screen is the same layers in another mode: set `data-modes="<collection id>:dark"` on the frame in HTML, or `props.modes` with `apply_transaction`. Don't redraw a screen in other colours.
- To show both, `duplicate_nodes` the screen and set the mode on the copy.
- `render` each mode you use: `render(documentId, nodeId, modes: "<collection id>:dark")`.

## 4. Repeated pieces are components

- A bar, panel, card or label that appears on more than one screen is a component (`create_component`), placed as instances. A fix to the component then reaches every screen.

## 5. Check before you say it's done

- `get_html` the frames you drew and scan for colour literals outside a `var(...)` fallback (`#rrggbb`, `rgb(`). Each one is either bound to a variable or deliberately part of the content (a photo-like gradient, a brand mark), and you can say which.
- `inspect` for `issues`; `render` and look at it, in every mode you defined.
- Tell the user the document name, what you built, and the variables you made or used.

## Common mistakes

- **"It's only a mockup."** Mockups are where drift starts. Bind anyway; it costs one call up front.
- **Naming by value** (`color/green`, `color/gray-800`). When the value changes the name lies. Name the role.
- **A second collection for dark.** Dark is a mode of the same collection, not a parallel set of variables.
- **The collection name used as an id.** Modes take `vc_…:<mode id>`, from `get_variables`.
- **Literal fallbacks that disagree with the variable.** Write the variable's current value as the fallback, so a broken binding still draws the right colour.
- **Bare text in a `<div>`** is refused (wrap it in `<p>`), and **stale reads**: use each result's `version` as the next `baseVersion`.
