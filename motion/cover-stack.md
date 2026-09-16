# Cover stack

A later opaque section slides over a pinned one. Not GSAP: `position: sticky` plus stacking order.

Shared rules: [MOTION.md](../MOTION.md).

Siblings sit in a pin wrapper:

```html
<div class="cover-pin">
  <section class="cover-sticky">...</section>
  <section class="cover-over">...</section>
</div>
```

The wrapper is the sticky containing block. It bounds how long the first section may stick. When `.cover-pin` leaves the viewport, the pinned section leaves with it. Quote, related, CTA, footer sit *after* the wrapper and scroll normally.

```css
.cover-pin {
  position: relative;
  isolation: isolate;
}

.cover-sticky {
  position: sticky;
  top: 0;
  z-index: var(--z-base);
}

.cover-over {
  position: relative;
  z-index: var(--z-header);
  background-color: var(--color-surface);
}
```

How it plays:

1. **`sticky; top: 0`** — The first section sticks to the viewport top while the rest keeps scrolling.
2. **Higher `z-index` on the opaque block** — The second section paints over the first and covers it as it arrives.
3. **`isolation: isolate`** — The stack stays inside the wrapper. Without it, the sticky layer can bleed through later footer or CTA.

Sticky does not collapse the first section's height. It still occupies its full layout size. Sticky only changes *where* it is painted, not *how much space* it takes. That is why the opaque block really slides over it, instead of starting underneath.

Cover with an opaque surface and an existing z-index token. A translucent overlay does not cover; it muddies.

## Reduced motion

Drop sticky so sections stack in document order:

```css
@media (prefers-reduced-motion: reduce) {
  .cover-sticky {
    position: relative;
  }
}
```

## Checklist

- [ ] Pin wrapper uses `isolation: isolate`
- [ ] Covering surface is opaque and a higher z-index token
- [ ] Content after the effect sits outside the pin wrapper
- [ ] `prefers-reduced-motion` restores normal document flow
