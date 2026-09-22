# Sanity

Checklist for projects using Sanity Studio or Sanity content.

Shared rules and checklist: [PRINCIPLES.md](../PRINCIPLES.md). Also walk [CMS checklist](cms.md).

---

## Dev Principles

- Default Studio preview: [`http://localhost:3333/`](http://localhost:3333/). Don't spawn a second Studio. If another project already owns that port, don't kill it: use the next free one, or the URL this Studio already printed. The frontend still uses its own default (Astro [`http://localhost:4321/`](http://localhost:4321/), Next.js [`http://localhost:3000/`](http://localhost:3000/)). See [Local preview](../PRINCIPLES.md#local-preview).

### Studio hosting

- Studio is its own package in `studio/` in the same git repo as the site. Do not create a studio with **+ Add studio** in Sanity Manage, and do not host the CMS on the public site domain.
- Self-host on a subdomain (`studio.example.com`). Second Vercel project, same repo: Root Directory `studio`, Framework Other, build `npm run build`, output `dist`.
- `studio/vercel.json` rewrites all paths to `index.html` (SPA).
- Deployment Protection off. Sanity login is the gate.
- After the first deploy, register the external URL (uploads nothing; tells Sanity where Studio lives):

```sh
npx sanity deploy --external --url https://studio.example.com
```

Repeat after schema changes so Dashboard and Media Library stay in sync. `studioHost` in `sanity.cli.ts` stores the production URL.

### CORS

- Allow credentials. Origins: `http://localhost:3333`, `http://localhost:4321` (or the Next dev origin), the production site (apex and `www` if both exist), and the Studio origin.
- Do not add `https://*.vercel.app` with credentials.

### Content model

- Pages that exist once are singletons: fixed `_id`, removed from "new document" templates, actions limited to publish, discard changes, and restore.
- Lists editors reorder use an orderable document list. Sort in the query matches that order.
- Structure groups singletons (pages, settings) separately from collections.
- A link field is an object: internal page reference or external URL (`https`, `mailto`, `tel`). The frontend query resolves it to one string.

### Images

- Alt text lives on the media asset (`altText`), set once in the media library, reused everywhere.
- The image input shows that asset alt. Validation fails when the asset has no alt. A per-field `alt` is only a legacy fallback: `coalesce(asset->altText, alt)` in GROQ.

### Editor tools

- Presentation is the preview tab. `defineDocuments` routes match public URLs, including slug filters.
- `defineLocations` lists where a document appears. Shared modules (settings, a logo bar, pricing) get a caution message that names the pages. Do not invent a single fake URL.
- Hide Vision and Releases from non-administrators.
- Add a Studio locale when editors do not work in English.
- Use `sanity-plugin-media` for the media library.

---

## Checklist

- [ ] Studio deployed and reachable
- [ ] Studio is on its own subdomain, registered with `sanity deploy --external`
- [ ] Deployment Protection is off
- [ ] CORS origins set for production + preview domains
- [ ] CORS credentials on for site + Studio origins; no wildcard `*.vercel.app` with credentials
- [ ] Preview / visual editing tested end-to-end
- [ ] Published vs draft content verified on production
- [ ] Image alt is stored on the asset and blocks publish when missing
- [ ] Singletons cannot be duplicated or deleted from the desk
- [ ] Non-admins do not see Vision or Releases

## Resources

- [`sanity-plugin-media`](https://www.sanity.io/plugins/sanity-plugin-media) – media browser
- [`@sanity/code-input`](https://www.sanity.io/plugins/code-input) – code editor with syntax highlighting
- [`next-sanity`](https://www.sanity.io/plugins/next-sanity) – official Next.js toolkit
- [`sanity-astro`](https://www.sanity.io/plugins/sanity-astro) – official Astro integration
- [Official Sanity plugins](https://www.sanity.io/exchange/type=plugins/by=sanity)
- [Sanity Recipes](https://www.sanity.io/recipes) – schema & code snippets
