---
name: review-and-fix
description: Find and fix layout problems in a Palet design (clipped or overflowing text, zero-size layers, missing fonts, broken variable bindings, overlaps). Use when the user asks to check, review, QA, clean up or fix screens in Palet, or after drawing something large.
---

If a Palet tool fails with an authentication error, tell the user to run `/mcp`, choose **palet** and sign in (no account: https://app.palet.so/sign-up). Don't ask for an API key.

## 1. Measure, then look

- `get_tree` to find the frames to review (or the ones the user names).
- `inspect(documentId, nodeId)` on each: every layer's `box`, how many `lines` each text wraps to, and `issues`. It's cheaper than an image; start here.
- `render` the frames with issues, and any the user cares about visually. `inspect` finds measurable problems; `render` finds the ones it can't (misalignment, uneven spacing, poor contrast).
- If the document has variables with modes, review each mode: `inspect` and `render` take `modes: "<collection id>:<mode id>"`.
- Look for hard-coded values: `read` the frames and find colours (and repeated sizes) written as literals (`bg-[#…]`, `style="color: …"`) instead of theme classes. A literal that equals a variable's value becomes that variable's class (`bg-primary`); a colour with no variable gets one first (`set_variables`, named by its role), then its class. If the document has no variables at all, propose a `Theme` collection (the design-on-canvas skill) before fixing anything else.

## 2. What each issue means

| Code | Usual fix |
|---|---|
| `text_overflow` | A fixed width or height is too small: make it `w-fit`, let it wrap, or widen the parent |
| `clipped` | A child sticks out of a frame that clips: resize it, or give the parent `overflow-visible` if it's meant to overhang |
| `truncated` | A line clamp hides text: raise the clamp, or shorten the copy if the user agrees |
| `zero_size` | No width or height: usually an empty flex child; give it a size or remove it |
| `font_missing` | The family isn't a Google font or is misspelled: use its exact name |
| `css_dropped` | A raw CSS declaration was refused: use a supported class instead |
| `overlap` | Two layers of the page sit over each other: move one with `left` / `top` in its `style` |
| `unresolved_variable` | The binding has no value in this mode, or names a deleted variable: `set_variables` the missing mode, or rebind |
| `missing_component` | The instance's component is gone: point it at another, or `detach_instance` |

## 3. Fix with the smallest write

- `read(documentId, nodeId)` for the frame (compact form: classes, instances by name with props), change only what's wrong, and send it back with `write`, using that `version` as `baseVersion` and `check: true`. Check the result's `ignored` for classes that did nothing.
- For one property on one layer, `apply_transaction` with an `update` op is smaller and safer.
- A prop on one instance (`<Row title="…"/>`) changes that instance only; fix the component's `<template data-component>` if every instance has the problem. An instance's `class` sets only its box.
- Repeat until `inspect` returns `issues: []` for every frame you touched, then `render` once more.

## 4. Report

Tell the user what you found, what you changed and what's left for them to decide (copy that doesn't fit, a design choice). Don't change content or visual design they didn't ask about.

## Common mistakes

- **Leaving layers out of `write`.** A layer the HTML leaves out is deleted (the call is refused with `would_delete` unless `allowDelete: true`). Send the frame's full HTML from `read`.
- **Sending stale HTML.** HTML read before a write lacks the ids that write created; sending it again duplicates them. Read again after every write.
- **Person-wins conflicts.** If someone edited the same property after your read, your change comes back in `skipped`. Read again and decide; don't force it.
- **Fixing `text_overflow` by shrinking the font.** Fix the box, not the type scale, unless the user asks.
