# Astro + Sanity

Stack addendum for content-driven Astro sites with Sanity Studio.

Shared rules and checklist: [PRINCIPLES.md](../PRINCIPLES.md). Walk **All projects** → **CMS** → **Sanity** → **Astro**.

Astro-specific rules + checklist: [astro.md](astro.md).
Sanity checklist: [sanity.md](sanity.md) (also walk [cms.md](cms.md)).

---

## Dev Principles

### Runtime

- Visual editing needs a server runtime. Set Astro `output: "server"` and an adapter. A static export cannot set the draft cookie.
- Two local processes: the site at [`http://localhost:4321/`](http://localhost:4321/) and Studio at [`http://localhost:3333/`](http://localhost:3333/). See [Local preview](../PRINCIPLES.md#local-preview).
- `useCdn: false` on the server client when a token or drafts are involved. The CDN does not apply tokens.

### Env

- `PUBLIC_SANITY_PROJECT_ID` and `PUBLIC_SANITY_DATASET` may be public.
- `SANITY_API_READ_TOKEN` is a **Viewer** token. Server only. Never prefix it with `PUBLIC_`.
- Create it with `npx sanity tokens create "<name>" --role=viewer --yes` from `studio/`.
- `PUBLIC_SANITY_STUDIO_URL` is the Studio origin used by stega (click-to-edit). Local default `http://localhost:3333`.
- Studio preview origin override: `SANITY_STUDIO_PREVIEW_URL` in the Studio project (local default `http://localhost:4321`).
- Document every key in `.env.example`. Never commit `.env`.

Verify anonymous published reads. If a document type is missing for anonymous clients, pass the Viewer token on the server for published queries too. Still never send that token to the browser.

### Code layout

Frontend data lives in `src/lib/sanity/`:

| File | Role |
|---|---|
| `fragments.ts` | Shared GROQ projections (images, links, portable text) |
| `queries.ts` | `defineQuery` strings + one fetcher per query |
| `load-query.ts` | Perspective, stega, source map, token |
| `draft-mode.ts` | Read the preview cookie |
| `image.ts` | `@sanity/image-url` helpers |
| `clean.ts` | `stegaClean` for hrefs, ids, comparisons |
| `index.ts` | Public exports |

Pages stay thin. Every fetcher takes `{ perspectiveCookie }` from `getDraftModeProps(Astro.cookies)`. Navbar, footer, and shared components that fetch must pass the same cookie, or the preview iframe shows published content.

### TypeGen

Configure typegen in `studio/sanity.cli.ts` so Studio generates frontend types:

- `generates`: `../src/lib/sanity/sanity.types.ts`
- `path`: `../src/**/*.{ts,tsx}` (where `defineQuery` lives)
- `overloadClientMethods: false` when a custom `loadQuery` is used

After a schema or GROQ change:

```sh
cd studio && npm run schema:extract && npm run typegen
```

Do not hand-edit `sanity.types.ts` or `schema.json`.

Queries use `defineQuery` from `groq`. Shared projections are string constants interpolated into those queries.

TypeGen inlines object properties. It does not inline functions. If one projection is reused with different field prefixes (`href`, `ctaHref`), precompute each variant as a const object property. Do not call a function inside the query string.

Fetcher return types come from the generated `*QueryResult` types. No `any`.

### Draft preview

Presentation tool:

- `previewUrl.previewMode.enable`: `/api/draft-mode/enable`
- `previewUrl.previewMode.disable`: `/api/draft-mode/disable`
- `allowOrigins`: localhost plus the real site origins (apex and `www` if both exist)

Enable route:

1. Require `SANITY_API_READ_TOKEN`.
2. `validatePreviewUrl` from `@sanity/preview-url-secret`. Reject invalid secrets.
3. Set the perspective cookie from `studioPreviewPerspective` (fallback `"drafts"`).

Cookie requirements (cross-site Studio iframe):

- `httpOnly: false` so visual editing can read it
- `sameSite: "none"`, `secure: true`, `path: "/"`
- `Partitioned` when the enable request is a cross-site iframe (`sec-fetch-dest: iframe` and `sec-fetch-site: cross-site`)

Disable route must expire **both** cookies: the partitioned one and the unpartitioned one (`Max-Age=0`, `SameSite=None`, `Secure`). Expiring only one leaves preview stuck.

`loadQuery`:

- No preview cookie → `perspective: "published"`, no stega, no source map, no token (unless anonymous reads are broken; see Env).
- Preview cookie → perspective from the cookie, `stega: true`, `resultSourceMap: "withKeyArraySelector"`, token required. Throw if the token is missing.

### Stega

Run `stegaClean` before a CMS string is used as an href, id, comparison, `<title>`, or meta description. Encoded characters break links and attributes. A small `cleanAttr` helper is enough.

### Framing and cache

- `Content-Security-Policy: frame-ancestors` allows `'self'`, the production Studio origin, and `http://localhost:3333`. Without this, Presentation shows a blank iframe.
- Do not cache draft HTML on the CDN. Skip public `Cache-Control` when the perspective cookie is set, or the edge can serve a draft to visitors.
- `/api/*` (enable/disable) is not cached.

### Visual editing client

Load `@sanity/visual-editing` only when the preview cookie is present.

- Turn off Astro `<ClientRouter />` in draft mode. View transitions fight the Studio history adapter.
- Studio-driven URL changes (`push` / `replace` / `pop`) use a full navigation, not the client router.
- On content refresh: fetch the current URL, patch `<img>` `src` / `srcset` / `alt` in place when the rest of the page is unchanged, otherwise hard-reload. Save scroll in `sessionStorage` and restore it after reload.
- Repeated uses of the same image need a stable `data-sanity-image` slot (field path). Otherwise overlays and in-place refresh cannot tell the copies apart.
- The "exit preview" control is `hidden` until script runs, and only when `window.self === window.top`. Inside the Studio iframe it stays hidden.

### Images

- Build URLs with `@sanity/image-url` and `.auto("format")`. One image component for CMS images.
- Allow `cdn.sanity.io` in the Astro image config.
- Project image fields (hotspot, crop, asset metadata: lqip, dimensions). Alt comes from the asset; see [sanity.md](sanity.md).

### Links

Internal vs external links are a `pageLink` object in the schema. GROQ resolves them to a string `href`. Components receive a URL, not a reference. Portable Text link marks use the same projection.

### Sharp

If `astro:assets` fails to load Sharp's native bindings, externalize `sharp` in Vite (`ssr.external` and `optimizeDeps.exclude`) for the server environments. Do not bundle it.

### Resources

- [`@sanity/astro`](https://www.sanity.io/plugins/sanity-astro)
- [`@sanity/preview-url-secret`](https://github.com/sanity-io/preview-url-secret)
- [`@sanity/visual-editing`](https://www.sanity.io/docs/visual-editing)
- [`astro-portabletext`](https://github.com/theisel/astro-portabletext) when body copy is Portable Text
- [Sanity Astro visual editing](https://www.sanity.io/docs/visual-editing/visual-editing-with-astro)

---

## Checklist

- [ ] Site is server-rendered; not a static export
- [ ] Viewer token is server-only and documented in `.env.example`
- [ ] `schema:extract` + `typegen` run after the last schema or GROQ edit; generated files are not hand-edited
- [ ] Draft iframe shows unpublished content; a normal visit shows published only
- [ ] Enable and disable both work from the hosted Studio (partitioned cookie cleared)
- [ ] Studio can frame the site (`frame-ancestors`); draft HTML is not CDN-cached
- [ ] Click-to-edit overlays land on the right field, including repeated images
- [ ] Links, ids, titles, and meta are stega-clean
- [ ] `<ClientRouter />` is off while preview is on
- [ ] Sanity CORS includes the site origins **with credentials**
