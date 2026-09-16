# Animation Principles

Catalog for scroll choreography and motion recipes. Build rules stay in [PRINCIPLES.md → Motion](PRINCIPLES.md#motion).

CSS first. Reach for [GSAP](https://gsap.com/docs/v3/) only when CSS can't (timeline, scrub, sequenced, interruptible, inertia).

One pattern per file in [`motion/`](motion/). Add a row here when you add a recipe.

| Pattern | Kind | File |
|---------|------|------|
| Cover stack | CSS sticky | [motion/cover-stack.md](motion/cover-stack.md) |
| Card stack | CSS sticky | [motion/card-stack.md](motion/card-stack.md) |
| Drag carousel | GSAP Draggable | [motion/drag-carousel.md](motion/drag-carousel.md) |

---

## Rules of thumb

- Pin wrappers bound how long something may stick. Never `sticky` a hero on `body`.
- Keep later page sections outside the pin wrapper.
- Use z-index tokens. Don't invent a one-off layer.
- Animate `transform` and `opacity` (GSAP: `autoAlpha`). Not layout props (`top`, `height`, `margin`).
- Pause looping motion on hover/focus and when off-screen.
- JS motion: `gsap.matchMedia()` for breakpoint and reduced-motion variants. Kill tweens, Draggables, and ScrollTriggers on cleanup.

---

## Checklist

Walk the pattern file for the effect you shipped, then:

- [ ] Effect is CSS unless a timeline, scrub, inertia, or interruptible sequence needs GSAP
- [ ] `prefers-reduced-motion` restores a usable non-motion path (document flow or native scroll-snap)
- [ ] JS motion is killed on page leave; observers disconnected
