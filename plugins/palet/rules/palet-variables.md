---
description: Every design written to Palet binds its colours, spacing, radii and type to Palet variables
alwaysApply: true
---

When you create or change a design in Palet (any `write_html`, `apply_transaction` or `duplicate_nodes` that draws), use variables. This includes quick mockups and explorations.

- Call `get_variables` before drawing. Reuse the document's collection if it has one; otherwise create a `Theme` collection with `light` and `dark` modes (`set_variables`) first.
- Bind every colour as `var(--cssName, #fallback)`, using the `cssName` that `get_variables` returns. Bind spacing, radii and font sizes you repeat the same way.
- Name variables by role (`color/text`, `color/surface`, `color/accent`), not by value (`color/green`).
- Make a dark screen by setting the frame's mode, not by redrawing it with other colours.
- Before reporting, `get_html` what you drew and make sure no colour is a bare literal outside a `var(...)` fallback.

The design-on-canvas skill has the full workflow.
