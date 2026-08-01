# Full component edit mode

Use this reference for decks declared with `data-template-edit-mode="components"`, including requests such as “每个元素都能改”, “像 PPT 一样拖动”, or “文字、卡片、图表都能调整”. Preserve the chosen preset's visual grammar while making authored semantic elements directly manipulable.

## 1. Split semantic content, not the rendering engine

Promote these authored items to independent `[data-slide-object][data-oid]` objects:

- headings, paragraphs, labels, metrics, list numbers, and table cells;
- meaningful cards, frames, icons, callouts, content shapes, images, and videos;
- chart titles, legends, editable values, value boxes, and annotations.

Keep these structural layers locked:

- full-slide backgrounds and page-level color fields;
- layout grids, axes, ticks, SVG paths, texture/glitch layers, masks, and animation wrappers;
- pure flex/grid containers whose only job is positioning children.

If a card contains editable text, create one graphic object for the empty card surface and separate text objects for its content. Exclude roots carrying `data-edit-slot` from graphic-replica capture so the same item never becomes both a text object and a duplicate graphic.

## 2. Componentize without layout collapse

Perform conversion in two phases for each slide:

1. Read and store every eligible source node's `getBoundingClientRect()`, computed typography, background, border, shadow, and z-order relative to the slide.
2. After all measurements are captured, move/wrap text nodes and create graphic replicas inside `.slide-edit-layer`.

Never measure and move one node at a time: removing the first flex/grid child can reflow later nodes and collapse their positions. Use percentages relative to the slide for `left`, `top`, `width`, and `height`.

Each wrapper must contain:

```html
<div class="slide-object" data-slide-object data-oid="deck-o17" data-object-type="text">
  <button class="slide-object-move" type="button" aria-label="Move object">⠿</button>
  <button class="slide-object-delete" type="button" aria-label="Delete object">×</button>
  <button class="slide-object-resize slide-object-resize--se" type="button" data-resize-handle="se" aria-label="Resize"></button>
  <div class="slide-object-text" contenteditable="false">Editable content</div>
</div>
```

Assign unique document-wide `data-oid` values after conversion. Mark source slots with `data-component-source-slot`; mark graphic replicas with `data-component-source-graphic`. Hide original graphic roots only after the replica exists. Give transparent/empty graphic replicas a nearly transparent edit-mode hit target so the user can select the card surface.

## 3. Make editing obvious

- Keep **Edit** and **Pages** visible at all times; do not require discovery of a hover corner.
- Let **E** toggle edit mode. Change the button label/state so entry and exit are unambiguous.
- Double-click any authored object in presentation mode to enter edit mode, select it, and focus text when applicable.
- Keep Undo, Redo, Done, Save, and Add element visible while editing.
- Do not automatically open the Pages sidebar on edit entry; preserve the full canvas until the user requests page management.
- Show a short, non-blocking edit hint: click text to type, drag **⠿** to move, drag handles to resize, and use **×** to delete.
- Pause entrance motion in edit mode. Show outlines and handles only in edit mode.

## 4. Keep navigation available while editing

Add explicit previous/next buttons and a current/total counter in addition to arrows, Space, wheel, and nav dots. Do not disable the page-turn controls in edit mode. Keep `deck.current`, `.is-active`, the counter, and the current slide's motion class synchronized after button, keyboard, scroll, reorder, copy, delete, or new-page actions.

Enumerate slides only from the real deck root:

```js
const slides = Array.from(deckRoot.querySelectorAll(':scope > section.slide'));
```

Never count filmstrip clones with a global `querySelectorAll('section.slide')`.

## 5. Give every page its own motion

Assign a semantic `data-motion` value per slide and animate direct slide objects with a stagger index such as `--motion-i`. On navigation:

1. remove the active motion class from every non-current slide;
2. force style recalculation for the current slide;
3. add the active motion class on the next animation frame.

Use one coherent entrance system per page (e.g. poster drop, split sides, card deal, chart rise), not random animation on every child. Disable motion under `body.deck-edit-mode` and `prefers-reduced-motion: reduce`.

## 6. Keep data graphics linked

Editable chart values must be text/metric objects, while axes and paths remain structural. Listen for `input` and restored-state changes, parse the visible value, clamp it to the chart domain, and update the corresponding bar height/position, line point, label, or table total. Verify the chart still synchronizes after save/load and exported HTML reopen.

## 7. Export the editor, not a screenshot

Export a standalone single HTML file containing the latest deck DOM, inline CSS/JS, embedded media, page navigation, editor runtime, motion definitions, and persisted state. Strip only transient state such as active selection, edit mode, open drawers, temporary handles added solely for interaction, and active laser state. Keep future editing available in the exported file.

## 8. Required QA

- Every real slide has at least one independent object; all `data-oid` values and slide ids are unique.
- Click Edit, modify a title, select a graphic, move one object, and resize it with each supported handle.
- Double-click an object from presentation mode and verify it becomes selected/editable.
- Navigate previous/next while edit mode remains active; counter and current slide stay correct.
- Visit every slide and verify its motion replays only on that slide.
- Change a chart value and verify the graphic updates.
- Open Pages, reorder, copy, add a new page, and delete a page without duplicate ids.
- Save, export HTML, reopen the exported file, and repeat a text edit.
- Check 1280×720 with no overflow and inspect browser warnings/errors.
