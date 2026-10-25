---
name: design-on-canvas
description: Draw anything in Palet (screens, mockups, flows, explorations, slides, UI states) with every colour, size and font bound to Palet variables. Use whenever you create or change a design in Palet and no more specific Palet skill applies, including quick mockups and throwaway explorations.
---

If a Palet tool fails with an authentication error, tell the user to run `/mcp`, choose **palet** and sign in (no account: https://app.palet.so/sign-up). Don't ask for an API key.

**Always use variables.** Every colour, and every spacing, radius and font value you use more than once, is a Palet variable, reached through a theme class (`bg-primary`, `gap-3`, `rounded-md`) or `prefix-(--name)`. This holds for a quick mockup too: designs that start as sketches become the reference, and a hard-coded colour drifts the moment two screens use it. With variables, a theme change, a dark version or a consistency fix is one edit instead of a redraw.

## 1. Variables before the first layer

- `get_variables(documentId)` first. If the document has a collection, **use it**: draw with its variables, and add only what is missing with `set_variables` (an existing name is updated, not duplicated).
- If it has none, make one with `set_variables(documentId, collection, modes, variables, id, baseVersion)` before drawing:
  - `collection: "Theme"`, `modes: ["light", "dark"]`, and a value for both modes, even if the user only asked for one look.
  - Name by role, not by colour: `color/background`, `color/surface`, `color/border`, `color/foreground`, `color/muted-foreground`, `color/primary`, `color/danger`. A colour with a meaning gets its own name (`color/selection`), so the meaning is visible in the inspector.
  - A second single-mode collection for numbers you repeat: `space/1…8`, `radius/sm|md|lg`, `text/sm|md|lg`, and a string for the font (`font/body`).
- If the codebase has tokens, take them from it (the code-to-design skill) instead of inventing a palette.

## 2. Write classes; the variables are the theme

`write_html` takes Tailwind v4 classes, and the document's variables are the theme:

| Variable | Classes |
|---|---|
| `color/primary` | `bg-primary` `text-primary` `border-primary` |
| `space/2` | `gap-2` `p-2` `m-2` |
| `radius/md` | `rounded-md` |
| `text/sm` | `text-sm` |
| `font/body` | `font-body` |
| a shadow variable, or any other | `shadow-(--shadow-card)`, `prefix-(--name)` |

- Other values are literals, and arbitrary ones work (`h-[52px]`). A new colour you need mid-design is a new variable first (`set_variables`), then its class. Never "just this once" as a literal, and never inline `style` or `var(--x, #fallback)`.
- `hover:`, `focus:`, `md:`, `dark:` and `transition-*` mean nothing on the canvas: they come back in `ignored`, never as an error.
- Icons are drawn by name: `<svg data-icon="mic" class="size-4"></svg>` (`search_icons` finds names). No separate `insert_icon`.
- A look repeated a few times, not worth a component: one root `<style>` block of simple class selectors (`.row`, `.row .title`; plain declarations, no states or media queries).
- Pass `check: true` and fix any `unresolved_variable`. A write that only creates layers can use `baseVersion: "latest"`, so a screen's sections can go in parallel.

## 3. Repeated pieces are components

A row, card, bar or label that appears more than once is a component, defined in the same write that places it:

```html
<template data-component="Row" class="flex items-center gap-3 h-11 px-4 rounded-md bg-surface">
  <svg data-prop="icon" data-icon="settings" class="size-4"></svg>
  <p data-prop="title" class="text-sm font-medium text-foreground">Title</p>
  <div data-prop="badge" class="px-2 rounded-full bg-primary"><p class="text-xs text-background">New</p></div>
</template>
<div data-name="Settings / General" class="flex flex-col gap-1 w-80 p-4 bg-background">
  <Row title="General" icon="settings"/>
  <Row title="Audio" icon="mic" badge="true"/>
</div>
```

- The template is drawn on the Components page. Its named layers (`data-prop` or `data-name`) are the props: a `<p>` takes text, any other layer takes `true`/`false` (visible), a nested instance swaps by component name, an icon swaps by icon name.
- An instance's `class` sets only its box (`w-full`, `grow`), as values. The result lists `components` ({id, name, props}) for reuse later.
- A fix to the component then reaches every screen.

## 4. Light and dark are modes, not copies

- A dark screen is the same layers in another mode: `data-modes="<collection id>:dark"` on the frame (collection id from `get_variables`), or `props.modes` with `apply_transaction`. Don't redraw a screen in other colours.
- To show both, `duplicate_nodes` the screen and set the mode on the copy.
- `render` each mode you use: `render(documentId, nodeId, modes: "<collection id>:dark")`.

## 5. Check before you say it's done

- `get_html` the frames you drew and scan for colour literals (`#rrggbb`, `rgb(`, `bg-[#…]`). Each one is either bound to a variable or deliberately part of the content (a photo-like gradient, a brand mark), and you can say which.
- Read the write's `ignored` list: it names every class and prop that did nothing.
- `inspect` for `issues`; `render` and look at it, in every mode you defined.
- Tell the user the document name, what you built, and the variables you made or used.

## Common mistakes

- **"It's only a mockup."** Mockups are where drift starts. Use the theme classes anyway; it costs one call up front.
- **Naming by value** (`color/green`, `color/gray-800`). When the value changes the name lies. Name the role.
- **A second collection for dark.** Dark is a mode of the same collection, not a parallel set of variables.
- **The collection name used as an id.** Modes take `vc_…:<mode id>`, from `get_variables`.
- **Writing the same row twelve times.** Define it once as a `<template data-component>` and place `<Row/>`.
- **Bare text in a `<div>`** is refused (wrap it in `<p>`), and **stale reads**: use each result's `version` as the next `baseVersion`.
