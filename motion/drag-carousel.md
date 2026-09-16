# Drag carousel

Horizontal card strip: drag, throw, snap. CSS cannot do inertia plus per-card distance feedback, so this one is GSAP.

Shared rules: [MOTION.md](../MOTION.md).

## Stack

- GSAP 3.15+ with [`Draggable`](https://gsap.com/docs/v3/Plugins/Draggable/) and [`InertiaPlugin`](https://gsap.com/docs/v3/Plugins/InertiaPlugin/)
- Vanilla TypeScript. Wire it from an Astro `<script>` (or any page that already loads GSAP).

```json
{ "gsap": "^3.15.0" }
```

## Markup

```html
<div
  class="drag-scroller"
  data-drag-scroller
  tabindex="0"
  role="region"
  aria-roledescription="carousel"
  aria-labelledby="section-heading"
>
  <p class="sr-only">Drag, swipe, or use the arrow keys to browse.</p>
  <ul data-drag-track class="drag-track">
    <li><!-- card 1 --></li>
    <li><!-- card 2 --></li>
  </ul>
</div>
```

If Lenis (or similar) is on the page, add `data-lenis-prevent-horizontal` on the scroller so vertical smooth-scroll does not steal the drag.

Cards may be links. A click on a *non-active* card must `preventDefault` and snap instead of navigating.

## CSS

GSAP owns `x` on the track. The scroller must not scroll natively while Draggable is active.

```css
.drag-scroller {
  overflow: hidden;
  cursor: grab;
  touch-action: pan-x;
  user-select: none;
}

.drag-track {
  display: flex;
  width: max-content;
  gap: var(--spacing-gutter);
  padding-inline: var(--spacing-page);
  margin: 0;
  list-style: none;
}

.drag-track > li {
  flex-shrink: 0;
}
```

## Behaviour

1. `Draggable.create(track, { type: "x", inertia: true })`.
2. Snap points are measured, not hardcoded: for each `<li>`, the `x` that left-aligns it to the track's `paddingInlineStart`.
3. Inertia `snap.x` returns the closest snap point to the throw's end value.
4. Every drag frame (`onDrag`, `onThrowUpdate`) sets each card's opacity and scale from distance to that left anchor: opacity `1 → 0.4` and scale `1 → 0.95` across 60% of the scroller width.
5. Click a non-active card: `snapToIndex(i)` with `expo.out`.
6. `ArrowLeft` / `ArrowRight` on the focused scroller decrement / increment `currentIndex` and snap.
7. `ResizeObserver` re-snaps to the current index at duration `0` when the viewport changes.
8. `prefers-reduced-motion: reduce`: skip Draggable. Native `overflow-x: auto` plus `scroll-snap-type: x mandatory` and `scroll-snap-align: center` on items.

```js
Draggable.create(track, {
  type: "x",
  inertia: true,
  cursor: "grab",
  activeCursor: "grabbing",
  edgeResistance: 0.65,
  allowNativeTouchScrolling: true,
  zIndexBoost: false,
  snap: {
    x: (endValue) => closestSnap(endValue),
  },
  onDrag: updateOpacities,
  onThrowUpdate: updateOpacities,
  onThrowComplete() {
    currentIndex = findClosestIndex();
    updateOpacities();
  },
});
```

## Key functions

**`getSnapPoints()`** — array of `x` values, one per card. Measure each item's left against the scroller, then subtract the current track `x` and the page padding so the result is a transform, not a layout position:

```js
const pad = parseFloat(getComputedStyle(track).paddingInlineStart) || 0;
const trackX = gsap.getProperty(track, "x");
const scrollerLeft = scroller.getBoundingClientRect().left;

slides.map((slide) => {
  const slideLeft = slide.getBoundingClientRect().left - scrollerLeft;
  return -(slideLeft - pad - trackX);
});
```

**`snapToIndex(i, duration)`** — clamp `i`, tween the track to that snap with `overwrite: true`, update opacities on the tween.

**`updateOpacities()`** — `anchorX = scroller.left + pagePadding`. Per card, `progress = min(|card.left - anchorX| / (scroller.width * 0.6), 1)`, then `opacity: 1 - progress * 0.6`, `scale: 1 - progress * 0.05`.

**`findClosestIndex()`** — card whose left edge is nearest `anchorX`. Use this after a throw; don't trust the snap loop's last `i` alone.

Init with `snapToIndex(0, 0)` so the first card is already at the padding, then `updateOpacities()`.

## Cleanup

Kill the Draggable instance, kill tweens on the track and slides, disconnect the `ResizeObserver`. Astro view transitions and client navigations will otherwise leave a live drag on a detached node.

## Checklist

- [ ] InertiaPlugin registered; Draggable `type: "x"` with `inertia: true`
- [ ] Snap points measured from layout (padding + item left), not magic numbers
- [ ] Distance feedback uses `transform` + `opacity`, not layout props
- [ ] Keyboard arrows work on a focusable region; non-active card click snaps instead of navigating
- [ ] Resize re-snaps to the current index
- [ ] Reduced motion uses native scroll-snap; Draggable is not created
- [ ] Draggable, tweens, and ResizeObserver are killed on page leave
