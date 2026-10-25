---
description: Every design written to Palet binds its colours, spacing, radii and type to Palet variables, through the theme classes
alwaysApply: true
---

When you create or change a design in Palet (any `write`, `apply_transaction` or `duplicate_nodes` that draws), use variables. This includes quick mockups and explorations.

- Call `get_variables` before drawing. Reuse the document's collection if it has one; otherwise create a `Theme` collection with `light` and `dark` modes (`set_variables`) first.
- The variables are the Tailwind theme of `write`: `color/primary` is `bg-primary`, `text-primary`, `border-primary`; `space/2` is `gap-2` and `p-2`; `radius/md` is `rounded-md`; `text/sm` is `text-sm`; `font/body` is `font-body`. A shadow or any other variable: `shadow-(--shadow-card)`, `prefix-(--name)`.
- Never write a literal colour (`bg-[#6e4fa0]`, `style="color: #fff"`), and no `var(--x, #fallback)`. A value you repeat is a variable first (`set_variables`), then a class.
- Name variables by role (`color/foreground`, `color/surface`, `color/primary`), not by value (`color/green`).
- Make a dark screen by setting the frame's mode (`data-modes`), not by redrawing it with other colours.
- Before reporting, `read` what you drew and make sure no colour is a bare literal.

The design-on-canvas skill has the full workflow.
