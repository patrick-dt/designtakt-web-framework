# Card stack

Several cards pin in the same column. Each new card covers the last.

Shared rules: [MOTION.md](../MOTION.md).

- Wrapper: `isolation: isolate`.
- Each card: `position: sticky` with a shared `top` (clear the header).
- Large travel gap between cards (`svh`) so the next card has room to move over the previous.
- Raise `z-index` per card (token or `--n` custom property).
- Extra padding at the bottom of the stack so the last card can settle before the section leaves.

```css
.card-stack {
  isolation: isolate;
  padding-bottom: 30svh;
}

.card {
  position: sticky;
  top: 6.5rem;
  z-index: var(--card-z, 1);
}

.card + .card {
  margin-top: 48svh;
}
```

## Reduced motion

Restore normal flow. Drop sticky, drop the travel gap, use ordinary spacing:

```css
@media (prefers-reduced-motion: reduce) {
  .card-stack {
    padding-bottom: 0;
    gap: 1.25rem;
  }

  .card {
    position: relative;
    top: auto;
  }

  .card + .card {
    margin-top: 0;
  }
}
```

## Checklist

- [ ] Wrapper isolates the stack
- [ ] Travel gaps are `svh`, not magic `px`
- [ ] `z-index` rises per card via a token or `--n`
- [ ] `prefers-reduced-motion` drops sticky and travel gaps
