# Discovery: Druxt frontend for the Drupal CMS (Haven) demo

Written for the design and frontend owners of the Drupal CMS demo.

Every answer is marked **confirmed** (observed in code or in a response),
**inferred** (reasoned from what was observed) or **unknown** (not established).
Sources are paths in this repository's installed codebase, or URLs.

## How this was checked

A fresh local install of this branch, following the README: `composer install`
then `.devtools/provision` on SQLite, served with the built-in server, and
queried anonymously. Drupal 11.4.9, 125 modules and themes enabled. The JSON:API
index lists 83 resource types, matching the live backend, so the local install
and the deployed environment agree.

Local-only changes made for inspection, none committed and all reverted:

| Change | Why | Reverted |
| --- | --- | --- |
| `drush role:perm:add anonymous 'administer components'` | Test whether the component registry opens | Yes |
| `drush role:perm:add anonymous 'administer patterns'` | Identify the binding access gate | Yes |

No production config was touched. The Druxt module needed no changes: it is
already enabled (see D.1), as are `jsonapi_views`, `jsonapi_menu_items` and
`decoupled_router`.

### Correction to the brief

The brief names the repository `druxt/drupal-cms.demo.druxtjs.org`. No such
repository exists. The work is in **`druxt/cms.demo.druxtjs.org`** (GitHub,
public; mirrored internally). Confirmed against both remotes. The same wrong
name appears in `AGENT_BRIEF.md` and `HANDOFF.md`.

---

## A. Haven theme: component contract

### A1. The SDCs — **confirmed**

26 components in `web/themes/contrib/haven_theme/components/`, each with a
`*.component.yml`. Full prop and slot detail is in the table at the end of this
document. Grouped by the `group` key:

- **Base** (8): `anchor`, `badge`, `blockquote`, `button`, `heading`, `icon`, `image`, `text`
- **Card** (6): `card`, `card-feature`, `card-icon`, `card-logo`, `card-pricing`, `card-testimonial`
- **Hero** (3): `cta`, `hero-billboard`, `hero-side-by-side`
- **Layout** (9): `accordion`, `accordion-container`, `footer`, `grid`, `group`, `navbar`, `section`

**No component sets `noUi`, and none sets `status` except six marked `stable`**
(`badge`, `button`, `grid`, `group`, `heading`, `icon`, `section`, `video`). The
rest leave `status` unset. So `noUi`/`status` do not usefully distinguish what
Canvas exposes; what Canvas exposes is the `component` config entity list (C11),
which has 50 entries against these 26 SDCs — the balance are block-derived.

**Eight of the 26 are never used in the seeded content** (A1 vs E): `anchor`,
`accordion`, `accordion-container`, `blockquote`, `card`, `card-feature`,
`card-pricing`, `cta`, `footer`, `navbar`, `video`.

### A2. Design tokens — **confirmed, and they do not match the design**

`web/themes/contrib/haven_theme/src/theme.css`. Haven's palette is **oklch**,
not hex, and its fonts are **Geist / Gelasio / Fira Mono**:

```css
--background: oklch(0.952 0 89.876);     --foreground: oklch(0.321 0 89.876);
--card: oklch(1 0 89.876);               --card-foreground: oklch(0.321 0 89.876);
--primary: oklch(0.292 0.057 154.902);   --primary-foreground: oklch(1 0 0);
--secondary: oklch(0.653 0.131 230.943); --secondary-foreground: oklch(1 0 89.876);
--muted: oklch(0.925 0 89.876);          --muted-foreground: oklch(0.321 0 89.876);
--accent: oklch(0.935 0.163 102.299);    --accent-foreground: oklch(0.321 0 89.876);
--destructive: oklch(0.6368 0.2078 25.3313);
--border: oklch(0.321 0 89.876);         --input: oklch(0.872 0.009 258.338);
--font-sans: "Geist", "Helvetica Neue", Arial, Helvetica, sans-serif;
--font-serif: "Gelasio", ui-serif, Georgia, Cambria, "Times New Roman", Times, serif;
--font-mono: "Fira Mono", "Menlo", "Consolas", "Liberation Mono", monospace;
--radius: 1rem;  --spacing: 0.25rem;  --tracking-normal: 0em;
```

Light and dark variants are derived, not hand-picked:

```css
--primary-light: color-mix(in oklch, var(--primary) 80%, white);
--primary-dark:  color-mix(in oklch, var(--primary) 80%, black);
--secondary-light: color-mix(in oklch, var(--secondary) 90%, white);
--secondary-dark:  color-mix(in oklch, var(--secondary) 85%, black);
--accent-light:  color-mix(in oklch, var(--accent) 95%, white);
--accent-dark:   color-mix(in oklch, var(--accent) 95%, black);
```

**`HANDOFF.md` specifies a different palette and different fonts** (`#FBF7F2`,
`#B84A2C`, Young Serif, Atkinson Hyperlegible, IBM Plex Mono). The *role names*
line up — `primary`, `secondary`, `accent`, `muted`, `white` — but no value
does. That is a re-skin, which is a legitimate choice, but it should be a
decided one. See "Decisions needed".

Haven also carries roles the design has no entry for: `card` /
`card-foreground`, `destructive` / `destructive-foreground`, `input`, and the
`*-light` / `*-dark` pairs. The navbar height is a token with three values
(see A6).

**Spacing is not a named step scale.** `--spacing: 0.25rem` is a base unit, and
the step values live in the component enums instead: `section.margin_block_start`
offers `-48 -20 0 8 20 32 48 64 96 128`, `margin_block_end` the same without the
negatives, and `padding_block_start` / `padding_block_end` only `0 16 32 64`.
`group.flex_gap` is `sm md lg xl` — **four steps, not three**; `HANDOFF.md` lists
`sm 8 / md 16 / lg 32` and omits `xl`. The pixel values behind `sm`–`xl` were not
located in the compiled CSS: **unknown**.

Heading sizes: `heading.text_size` enum is `default` plus
`heading-responsive-lg`, `xl`, `2xl`, `3xl`, `4xl`, `5xl`, `6xl`, `7xl`, `8xl`.
**`HANDOFF.md` covers 2xl–7xl only; `lg`, `xl` and `8xl` are missing from it.**
The px values per step were not located: **unknown**.

### A3. Breakpoints — **confirmed**

There is **no `haven_theme.breakpoints.yml`**. From the media queries in
`build/main.min.css`, with occurrence counts:

| Query | px | Uses |
| --- | --- | --- |
| `min-width: 40rem` | 640 | 4 |
| `min-width: 48rem` | 768 | 29 |
| `min-width: 64rem` | 1024 | 11 |
| `min-width: 80rem` | 1280 | 12 |
| `min-width: 96rem` | 1536 | 6 |

Plus `(hover: hover)` x4. This is Tailwind's default scale, mobile-first. The
dominant switch is **768px**, and `navbar.js:30` keys its desktop check to
`(min-width: 48rem)`.

**The design's 720px breakpoint does not exist in Haven.** `HANDOFF.md` and
`Canvas Notes.dc.html` both say columns collapse at 720px; the real switch is
768px.

### A4. Does `button` have a URL prop — **confirmed: yes, it does**

`components/button/button.component.yml:38-41`:

```yaml
    href:
      title: link
      type: string
      format: uri-reference
```

**This contradicts the premise in `HANDOFF.md` open issue 1, `AGENT_BRIEF.md`
item 9 and `Canvas Notes.dc.html`**, all of which say Haven's button has no link
prop and suggest raising it upstream. It has one. Nothing needs to change in
Haven.

What is true is that the *seeded content* mostly leaves it empty. Across all
seven Canvas pages there are 10 button instances and exactly **one** sets `href`:

| Page | Label | href |
| --- | --- | --- |
| Page not found | Go back home | `/` |
| Blog | Donate now | *(none)* |
| Blog | Learn more | *(none)* |
| About us | Donate now | *(none)* |
| About us | Learn more | *(none)* |
| Projects | Donate now | *(none)* |
| Projects | Learn more | *(none)* |
| Home | Read article | *(none)* |
| Home | Read article | *(none)* |
| Home | More about this project | *(none)* |

So this is a **content defect in Haven's sample content**, not a schema gap. The
fix is to seed the links upstream, or set them in this demo's own content. The
frontend should render `<a>` when `href` is present and `<button>` otherwise,
exactly as `HANDOFF.md` already says.

Seven other components expose links: `heading.url`, `card.url`,
`card-feature.url`, `card-icon.url`, `card-logo.url`, `image.url`,
`card-pricing.button_url`, plus `blockquote.cite_url` and
`card-testimonial.cite_url`.

### A5. The icon set — **confirmed**

`haven_theme.icons.yml`: **Phosphor, version 2.1.1**, MIT, from
`github.com/phosphor-icons/core`. Icons are **bundled as 1512 local SVG files**
in `icons/phosphor/{icon_id}.svg`, not loaded as a font or from a CDN. The
wrapper template uses `viewBox="0 0 256 256"` and `fill="currentColor"`, with
`size` an integer 16–96 in multiples of 4, default 32, and `alt` optional
(empty means `aria-hidden="true"`).

A `phosphor-vue` dependency in the frontend must be pinned to the same 2.1.1
icon set, or names will drift.

### A6. Header and navbar with `overlap_navbar` — **partial**

**Confirmed:** `hero-billboard` has `overlap_navbar` (boolean, default `false`)
and `overlay_opacity` with enum `0% 20% 40% 60% 75%` — note **`HANDOFF.md` says
"overlay 0–40%", but 60% and 75% also exist**. Height enum is `full large ribbon`
(default `full`). `flex_position` is `top-left center-left bottom-left
hero-center`.

**Confirmed:** `--navbar-height` is declared three times in `theme.css`, as
`calc(var(--spacing) * 14 | 20 | 22)` = **56px, 80px, 88px** at successive
breakpoints. The design's "88 desktop / 68 mobile" matches the desktop value but
not the mobile one.

**Confirmed:** `components/navbar/navbar.js` is a `ComponentInstance` that wires
a hamburger and close button, tracks `isOpen`, resets
`dropdown-menu__expand-button--has-been-opened` state on open, calls
`measureScrollbarAndObserve` on the dropdown, and attaches a scroll handler only
below `(min-width: 48rem)` and only while the navbar is still on screen, for
mobile menu positioning.

**Unknown:** the transparent-over-hero state and the solid-after-80px-scroll
behaviour described in `HANDOFF.md` were not located in the theme's JS or CSS.
The scroll handler that exists is for mobile menu positioning, not for a
transparent-to-solid transition. Treat that transition as **a design addition**,
not a Haven behaviour, until proven otherwise.

### A7. Components that use JavaScript — **confirmed (list), partial (behaviour)**

Five component-level scripts:

| Component | File | What it does |
| --- | --- | --- |
| `navbar` | `components/navbar/navbar.js` | Mobile menu open/close, scroll tracking below 768px |
| `accordion` | `components/accordion/accordion.js` | Not read |
| `accordion-container` | `components/accordion-container/accordion-container.js` | Not read |
| `video` | `components/video/video.js` | Not read |
| `anchor` | `components/anchor/anchor.js` | Not read |

Theme-level libraries add `lib/messages.js` (depends on `core/drupal`,
`core/once`) and `build/motion.js` (minified, `core/once`) — the latter backs
`group`'s `animation`, `animation_duration` and `animation_delay` props.

**There is no carousel component.** The behaviour of accordion, video and anchor
is **unknown**; all three are unused in seeded content, so none blocks the build.

---

## B. Canvas: data model

### B1. `canvas_page.components[]` — **confirmed, with two corrections**

Each entry has exactly these keys:

```
uuid, parent_uuid, slot, component_id, component_version, inputs, inputs_resolved, label
```

**Correction 1: `label` exists and is not mentioned anywhere in the design
notes.** It is the editor-facing name of the placed component.

**Correction 2: `inputs` is a JSON *string*, not an object.** `inputs_resolved`
is a parsed object. The design notes describe both as objects. A frontend must
`JSON.parse(inputs)` or simply ignore it and use `inputs_resolved`.

Meanings:

- `uuid` — the placed instance's id. `parent_uuid` — its parent instance, empty at root.
- `slot` — which named slot of the parent it sits in. Order within a slot is array order.
- `component_id` — `sdc.<theme>.<name>` or `block.views_block.<view>-<display>`.
- `component_version` — a 16-hex-character content hash of the component's schema, e.g. `sdc.haven_theme.heading` is `f463a8803de8d3f1`. Stable per schema, changes when the schema changes.
- `inputs` — the stored values, as authored, with entity references unresolved: `{"media":{"target_id":"20"}}`.
- `inputs_resolved` — the same values with references expanded.

`inputs_resolved` is **computed server-side and is stable for anonymous
JSON:API**: all seven pages return it in full with no authentication (verified
anonymously). Media expands to `{src, alt, width, height}` — see C10 for the
important detail in `src`.

Example, a `hero-billboard` on Home:

```json
"inputs": "{\"height\":\"large\",\"flex_position\":\"hero-center\",\"content_background\":false,
            \"overlay_opacity\":\"20%\",\"object_position\":\"center\",
            \"media\":{\"target_id\":\"20\"},\"overlap_navbar\":true}"
"inputs_resolved": {"media": {"src": "/sites/default/files/2026-03/....avif?alternateWidths=...",
                              "alt": "Test", "width": 1200, "height": 900}, ...}
```

### B2. Content templates and node rendering — **confirmed, and this changes the plan**

**Nodes are not Canvas trees.** The only component-tree field in the entire site
is `canvas_page.components`:

```
canvas field type id: component_tree
canvas field: canvas_page.components bundles=canvas_page
```

On all four node bundles, `field_content` is **`text_long`**, a rich-text body —
not a component tree. Field types per bundle are in C2.

So:

- JSON:API **cannot** return a resolved component tree for a node, because nodes have no tree. **Confirmed.**
- The 7 `content_template` config entities describe how Canvas *renders* a node, and are withheld from anonymous users (C11).
- A decoupled frontend can render node pages **straight from the JSON:API fields**, with its own layout, and needs Canvas for nothing. **Inferred, but firmly: there is no node tree to fetch.**

This resolves `HANDOFF.md` open issue 3 and item 5 of its rendering model: build
node pages as Druxt entity displays over the plain fields. That is not a stopgap,
it is the only option.

The dynamic prop source syntax used inside `content_template` to bind fields to
props was **not** read, because the entities are withheld. **Unknown.**

### B3. `page_region` — **confirmed: there are none**

`/jsonapi/page_region/page_region` returns `data: []` with **zero omitted
entries** — distinct from the withheld resources, which report omissions. So no
`page_region` entities exist; header and footer are **not** Canvas trees in this
install.

Header and footer therefore come from Drupal's block layout and the menus (C4).
The frontend must build both itself from the menus. **Inferred from the absence.**

### B4. Patterns — **partial**

One `pattern` entity exists and is **withheld** from anonymous users (1 omitted).
Whether patterns are inlined into pages or referenced could not be read from the
entity. From the page data, no component carries a pattern reference and every
tree is fully expanded, so patterns appear to be **inlined at authoring time**,
leaving no trace in `components[]`. **Inferred.**

### B5. Versioning — **partial**

`component_version` is a 16-hex content hash carried on every placed instance.
Observed values are one per `component_id` across all seven pages, so nothing in
the seeded content is on a stale version. What Canvas does to a page's `inputs`
when a component's schema changes was **not** tested, and is **unknown**. The
hash is the mechanism by which a mismatch could be detected by a frontend.

### B6. Preview — **unknown**

Not investigated. No decoupled preview configuration was found, and the External
JavaScript Components module is not installed. Note that `js_component` exists
as a resource type with one entity, withheld.

---

## C. Data available to a decoupled frontend

The access map, taken anonymously against a fresh install. "omitted" is the
count of entities JSON:API withheld.

| Resource | HTTP | readable | omitted | Verdict |
| --- | --- | --- | --- | --- |
| `canvas_page/canvas_page` | 200 | **7** | 0 | Fully open |
| `node/project` | 200 | **3** | 0 | Fully open |
| `node/blog` | 200 | **3** | 0 | Fully open |
| `node/person_profile` | 200 | **1** | 0 | Fully open |
| `node/page` | 200 | **1** | 1 | One withheld (unpublished) |
| `menu_link_content` | 200 | **18** | 0 | Fully open |
| `file/file` | 200 | **24** | 0 | Fully open |
| `media/image` | 200 | **22** | 0 | Fully open |
| `view/view` | 200 | **21** | 0 | Fully open |
| `component/component` | 200 | 0 | **50** | Withheld |
| `content_template` | 200 | 0 | **7** | Withheld |
| `pattern/pattern` | 200 | 0 | **1** | Withheld |
| `brand_kit/brand_kit` | 200 | 0 | **1** | Withheld |
| `js_component` | 200 | 0 | **1** | Withheld |
| `crop/focal_point` | 200 | 0 | **22** | Withheld |
| `webform/webform` | 200 | 0 | **2** | Withheld |
| `image_style` | 200 | 0 | **50** | Withheld |
| `responsive_image_style` | 200 | 0 | **29** | Withheld |
| `klaro_app` | 200 | 0 | **29** | Withheld |
| `klaro_purpose` | 200 | 0 | **7** | Withheld |
| `search_api_index` | 200 | 0 | **1** | Withheld |
| `page_region` | 200 | 0 | 0 | **None exist** |

The short version: **Canvas pages, nodes, menus, media and views are open.
Nearly everything else a frontend needs is withheld.**

### C1. Home, and `/` → `/home` — **confirmed**

`system.site:page` is `{"front": "/home", "404": "/404", "403": "/user/login"}`.
`decoupled_router` resolves it already:

```
/      -> entity: canvas_page canvas_page | label: Home | resolved: .../home
/home  -> entity: canvas_page canvas_page | label: Home | resolved: .../home
/about -> entity: canvas_page canvas_page | label: About us | resolved: .../about
```

So the frontend needs no special case for the front page. Home is a
`canvas_page` with **43 components**, anonymously readable in full.

### C2. Node bundles — **confirmed**

All four share: `title`, `field_content` (text_long), `field_description`
(string_long), `field_seo_title`, `field_seo_description`, `field_seo_image`,
`field_featured_image`, `metatag`, `path`, `moderation_state`, `publish_on`,
`unpublish_on`, `sae_exclude`, plus `field_seo_analysis` (yoast_seo, not exposed
in the JSON:API attribute list).

Bundle-specific:

- **project**: `field_featured` (boolean), `field_introduction` (text_long), `field_thumbnail` (entity_reference), `field_tags`
- **blog**: `field_blog__byline` (string), `field_tags`
- **person_profile**: `field_role`, `field_badge`, `field_callout_heading`, `field_callout_content`, `field_sidebar`
- **page**: `field_tags`

Path aliases are present on every node and are date-nested for project and blog
(`/projects/2026-10/...`, `/blog/2026-10/...`) but flat for people
(`/dr-alice-grossmann`). **The date in the alias means aliases change if content
is re-seeded with different dates** — resolve by path from the menu and view
data, never hard-code.

**Focal point is not available.** `crop/focal_point` withholds all 22 entities,
so the `object-position` behaviour `HANDOFF.md` specifies for card images cannot
be driven from Drupal as things stand. See the decisions list.

### C3. Views — **confirmed**

Two views, each with more displays than the design assumed:

| View | Displays |
| --- | --- |
| `blog` | `default`, **`all`** (block), **`latest`** (block), `related` (block), `rss` (feed) |
| `projects` | `default`, **`all`** (block), **`featured`** (block), `latest` (block), `related` (block), `rss` (feed) |

**There are no page displays** — every listing is a block, which is why they
appear in Canvas trees as `block.views_block.<view>-<display>`.

Four are used in seeded pages: `blog-all`, `blog-latest`, `projects-all`,
`projects-featured`. **`blog-latest` is missing from the `HANDOFF.md` registry
table.** `related` displays on both views are unused but available, and are the
obvious source for "related items" on node pages.

`jsonapi_views` is enabled, so these are reachable as
`/jsonapi/views/<view>/<display>`. Pager, items-per-page and exposed filters per
display were **not** read: **unknown**.

### C4. Menus — **confirmed, with one addition**

| Menu | Label | Links |
| --- | --- | --- |
| `main` | Main navigation | 3 |
| `secondary` | Secondary navigation | 2 |
| `footer` | Footer | 3 |
| `social` | Social | 3 |
| **`top-tasks`** | **Top tasks** | **7** |

18 `menu_link_content` entities, all anonymously readable, and
`jsonapi_menu_items` is enabled for resolved trees. **`top-tasks` with 7 links is
not mentioned in any design document** and is the largest menu in the site.

### C5. Webforms — **blocked**

Both `newsletter_signup` and `contact_form` exist but are **withheld** (2
omitted), so element definitions are not anonymously readable. Element
definitions: **unknown**.

More importantly, **there is no decoupled submission path installed**:
`webform_rest` is not present, nor `simple_oauth`, nor `consumers`. A decoupled
client cannot submit either form as the site stands. CSRF, auth and error shape:
**unknown**, because no endpoint exists to describe.

### C6. The donation add-on — **confirmed: none is installed**

No module matching `donat`, `stripe` or `commerce` is enabled. There is no
donation backend at all. The "Donate now" buttons in the seeded content have no
`href` (A4) and nothing to post to. The demo must simulate it, as `HANDOFF.md`
already plans.

### C7. Search — **partial**

`search_api`, `search_api_db` and `search_api_exclude` are enabled, and one
`search_api_index` exists but is **withheld**, so its fields are not readable
anonymously: **unknown**.

**There is no JSON:API search endpoint.** `jsonapi_search_api` is not installed.
Something must be added for decoupled search.

### C8. Klaro — **blocked**

`klaro` is enabled, with 29 `klaro_app` and 7 `klaro_purpose` entities, **all
withheld**. A frontend cannot read the cookie-consent configuration as things
stand. Config a frontend can read: **none**.

### C9. `brand_kit` — **blocked**

One entity, **withheld**. Contents **unknown**.

### C10. Image styles and `srcset` — **confirmed, and better than expected**

`image_style` (50) and `responsive_image_style` (29) are both **withheld**, so a
frontend cannot enumerate them. But it does not need to: Canvas puts the
responsive recipe directly in `inputs_resolved`.

```
"src": "/sites/default/files/2026-03/fakhrir-amrullah-FXMZncDmgNc-unsplash.avif
        ?alternateWidths=/sites/default/files/styles/canvas_parametrized_width--%7Bwidth%7D
         /public/2026-03/fakhrir-amrullah-FXMZncDmgNc-unsplash.avif%3Fitok%3DEei_vaen"
```

The `alternateWidths` query parameter is a **URL template with a `{width}`
placeholder** (URL-encoded as `%7Bwidth%7D`), pointing at the
`canvas_parametrized_width--{width}` image style, with the derivative's `itok`
already included. A frontend builds `srcset` by substituting widths into that
template. `width` and `height` come alongside, so intrinsic ratio is known
without a second request. Source files are AVIF.

This is the single most useful thing found for the design, and it is not
mentioned in any design document.

### C11. Exposing the component registry — **confirmed, and the obvious fix does not work**

The gate is `CanvasUiAccessCheck`, reached through
`ContentCreatorVisibleCanvasConfigEntityAccessControlHandler`:

- `web/modules/contrib/canvas/src/EntityHandlers/ContentCreatorVisibleCanvasConfigEntityAccessControlHandler.php:43-55` — `view` is `canvasUiAccessCheck->access($account)` AND `$entity->status()`.
- `web/modules/contrib/canvas/src/Access/CanvasUiAccessCheck.php:39-67` — allowed if the account holds **any** of `administer content templates`, `administer code components`, `administer brand kit`, `administer page template`, `administer patterns`, or has create/edit access to at least one content entity type with a Canvas field.

Two results from testing this directly:

1. **Granting `administer components` does nothing.** The registry's own
   permission is *not* one of the six the check consults. Verified: after
   granting it to anonymous, the response was still `data=0, omitted=50`.
2. **Granting any one of the other five opens all four resources at once.** With
   `administer patterns` on anonymous: `component` 50, `content_template` 7,
   `pattern` 1, `brand_kit` 1, all `omitted=0`.

Both changes were reverted.

**There is no read-only permission in Canvas.** Every relevant permission is an
`administer *` permission, which in the Canvas UI confers write access. So:

- **Not acceptable:** granting anonymous any of those five permissions. It would open editing, not just reading.
- **Recommended:** a small module in this repository implementing `hook_entity_access()` (or an `EntityAccessControlHandler` decorator) that returns `AccessResult::allowed()` for the `view` operation on `component`, and optionally `content_template`, `pattern` and `brand_kit`, when `$entity->status()` is true — read-only, no permission granted. Cache it on the entity, as the core handler does.
- **The Druxt module cannot do this today** (D1), so the hook belongs either in this demo repository or in a new release of `drupal/druxt`.
- **Upstream:** Canvas should offer a `view components` permission, or expose the registry read-only. See the upstream list.

---

## D. Druxt: capability gaps

### D1. What `drupal/druxt` exposes — **confirmed**

Version **1.3.1**, `core_version_requirement: ^10 || ^11 || ^12`
(`web/modules/contrib/druxt/druxt.info.yml:5,14`). It is already enabled, with
`jsonapi_views`, `jsonapi_menu_items` and `decoupled_router`.

Its whole surface:

- `src/EventSubscriber/ContactPathTranslatorSubscriber.php`, `ViewsPathTranslatorSubscriber.php`, `WildcardPathTranslatorSubscriber.php` — extend `decoupled_router` so contact forms, views pages and wildcard routes resolve.
- `src/Plugin/Condition/DruxtRequestPath.php` — a block visibility condition for decoupled paths.
- `src/Plugin/Validation/Constraint/DruxtResourceConstraint*.php` — validation.
- `src/Form/DruxtSettingsForm.php` at `/admin/config/services/druxt`, permission `administer druxt`.

**It exposes no schema endpoints and no component registry.** Schema in Druxt is
a frontend concern: the `druxt-schema` JS package reads
`entity_view_display` / `entity_form_display` over JSON:API at build time. So the
brief's framing — "prefer exposing through the Druxt module" — would require new
code in the module, not configuration. **Confirmed.**

### D2. Can `druxt-router` resolve a `canvas_page` — **confirmed: yes**

`decoupled_router` already returns, for `/`, `/home` and `/about`:

```
entity: canvas_page  bundle: canvas_page  label: Home
```

`druxt-router` consumes `decoupled_router` output, so it will see entity type
`canvas_page`, bundle `canvas_page`, and the matching JSON:API resource
`canvas_page--canvas_page`. No router work is needed to reach these pages.

What `druxt-router` will *not* do is render them: there is no Druxt component for
a component tree. **Inferred:** `DruxtRouter` will resolve and then fall through
to `DruxtEntity`, which has no display for `canvas_page`.

### D3. Does any Druxt package render Canvas trees — **confirmed: no**

No package in the `druxt.js` monorepo (`druxt`, `blocks`, `breadcrumb`, `entity`,
`menu`, `router`, `schema`, `site`, `views`) addresses component trees; they are
built around entity view modes, views, menus and blocks.

**Recommendation (inferred):** build `CanvasTree` and `CanvasComponent` **in the
demo repository first**, not as a `druxt-canvas` package. Two reasons from this
discovery: the tree rendering is small (parent/slot grouping plus a registry
lookup), and the hard part — the component registry and the prop contract — is
blocked on C11 and may change. Extract a package once the registry is reachable
and a second site needs it.

### D4. Naming — **inferred**

Druxt's pattern is `Druxt<Module><Thing>` with a wrapper/slot override chain:
`DruxtEntity`, `DruxtView`, `DruxtMenu`, `DruxtBlock`, each resolving a themeable
component by a list of suggestions from most to least specific.

Proposed, following it:

| Role | Name | Suggestion order |
| --- | --- | --- |
| Tree root | `DruxtCanvasPage` | `DruxtCanvasPage[Title]`, `DruxtCanvasPage` |
| One placed component | `DruxtCanvasComponent` | by `component_id`, then by source, then default |
| Per-SDC wrapper | `DruxtCanvasHavenHeroBillboard` … | derived from `sdc.haven_theme.hero-billboard` |
| Views block | reuse `DruxtView` | `block.views_block.<view>-<display>` maps to view + display |
| Unmatched | `DruxtCanvasFallback` | |

The `component_id` to component-name mapping is mechanical:
`sdc.haven_theme.hero-billboard` → `DruxtCanvasHavenHeroBillboard`.

### D5. The Umami demo bar and inspector — **unknown**

Not inspected. `druxt/umami.demo.druxtjs.org@develop` carries
`nuxt/components/app/DemoBar.vue`, `DruxtInspector.vue` and
`plugins/druxt-inspector.client.js`, and the design already specifies the Canvas
label scheme in `HANDOFF.md`. Whether the inspector can label Canvas components
unchanged was not established. The one concrete requirement this discovery adds:
labels need `component_id` **and** `component_version`, and the `label` key found
in B1 is a ready-made human name for the overlay.

### D6. Deployment conventions — **confirmed**

Shared with the sibling demos: one Lagoon environment per demo with `cli`,
`nginx`, `php` and MariaDB on pinned images; a post-rollout `drupal install`
from the site template; a `snapshot` after it; and a daily cron reset at 16:00
UTC. `DRUPAL_ACCOUNT_PASS` is set on the Lagoon project to give the admin a
known password.

This repository **does not** carry the `drush user:login` post-rollout task that
the Umami and Commerce demos had, and its README already documents
`DRUPAL_ACCOUNT_PASS`. It never had that disclosure problem.

The Lagoon project is named `demo-api-druxtjs-org` and keeps that name until it
can be renamed; the retired `demo-api.druxtjs.org` domain is now served by the
Umami backend, so nothing is stranded.

---

## E. Sample content inventory

Seven `canvas_page` entities and eight nodes. Per page: component count, image
count, the distinct image aspect ratios present, and the longest heading and
longest paragraph by character count.

| Page | Path | Entity | Comps | Imgs | Ratios | Longest heading | Longest para |
| --- | --- | --- | ---: | ---: | --- | ---: | ---: |
| Home | `/home` | canvas_page | 43 | 12 | 3:2, 1:1, 4:3, 16:9ish, 3.84:1, 1.53:1 | **100** | 371 |
| Leadership | `/leadership` | canvas_page | 27 | 5 | 1.54:1, 1.78:1, 1:1, 4:5 | 82 | 345 |
| About us | `/about` | canvas_page | 22 | 3 | 3:2 | **176** | **513** |
| Projects | `/projects` | canvas_page | 17 | 1 | 1.34:1 | **168** | 154 |
| Blog | `/blog` | canvas_page | 16 | 1 | 3:2 | **205** | 138 |
| Careers | `/careers` | canvas_page | 16 | 5 | 3:2, 1.53:1, 1:1 | 52 | 426 |
| Page not found | `/404` | canvas_page | 5 | 0 | — | **0** | 45 |

**The headings are the finding here.** The longest is **205 characters** (Blog),
with three more over 100. These are paragraphs placed in `heading` components,
not headlines. At `heading-responsive-7xl` (the design's 72px) a 205-character
heading is unusable. Either the design caps heading sizes by length, or the
content is re-authored. The 404 page has **no heading component at all**, only
text — the design assumes one.

Nodes:

| Node | Path | Bundle | Title | Body | Desc | Featured image |
| --- | --- | --- | ---: | ---: | ---: | --- |
| Rewilding the Greenway corridor | `/projects/2026-10/rewilding-greenway-corridor` | project | 31 | 2678 | 165 | 3:2ish |
| Solarizing the south side | `/projects/2026-10/solarizing-south-side` | project | 25 | 2576 | 192 | 3:2ish |
| Restoring the outer reef | `/projects/2026-10/restoring-outer-reef` | project | 24 | 3197 | 228 | 3:2ish |
| What the reefs are telling us | `/blog/2026-10/what-reefs-are-telling-us` | blog | 29 | 2828 | 158 | 1.26:1 / 3:2 / 1.79:1 |
| Six months without a car: what actually changed | `/blog/2026-10/six-months-without-car-what-actually-changed` | blog | 47 | 2221 | 139 | as above |
| What your local council can do about climate change right now | `/blog/2026-10/what-your-local-council-can-do-about-climate-change-right-now` | blog | **61** | 2218 | 132 | as above |
| Dr. Alice Grossmann | `/dr-alice-grossmann` | person_profile | 19 | 1109 | 0 | **4:5** |
| Terms of service | `/terms-service` | page | 16 | **31** | 42 | none |

Notes for the design:

- Project and blog bodies run **2.2k–3.2k characters** of rich text, so node pages need real long-form typography, not a teaser layout.
- **Terms of service has a 31-character body.** It is a stub.
- Dr. Alice Grossmann is the **only** person, at **4:5** portrait — the design's "1:1 circle portrait" would crop it.
- Blog titles reach 61 characters; card layouts must take three lines.
- Image ratios across the site are **not consistent**: 1:1, 4:5, 1.26:1, 1.34:1, 3:2, 4:3, 16:9 and a 3.84:1 banner all appear. Cards cannot assume one ratio, and with focal point withheld (C2) there is no crop hint.

### Image alt text quality — **confirmed**

Across the Canvas pages, 15 distinct alt strings. Most are good and descriptive,
but:

| Count | Alt |
| ---: | --- |
| 5 | `Placeholder brand logo` |
| 4 | `Placeholder` |
| 2 | **`Test`** |

The two `Test` alts are on the **Home hero image**, the most prominent image on
the site. The design relies on `inputs_resolved.alt`; on Home it would render
`alt="Test"`.

---

## Component table

Every `component_id` found in seeded content, with its props as used and its
schema contract. "JS" marks a component with a script.

| component_id | Used | Key props (type / enum) | Slots | JS | Notes |
| --- | ---: | --- | --- | :-: | --- |
| `sdc.haven_theme.heading` | 31 | `heading_text` str; `level` 1–6; `text_size` default + `heading-responsive-lg…8xl`; `text_color` default/inverted/secondary/accent/text-default; `align` left/center/right; `url` str | — | | `level` and `text_size` are independent |
| `sdc.haven_theme.group` | 23 | `flex_direction` column/row; `flex_gap` sm/md/lg/**xl**; `items_align`, `flex_align` start/center/end; `radius`, `padding` sm–xl; `background` primary/secondary/accent/white/muted; `stretch` bool; `animation` fade_up/down/left/right; `animation_duration` 150/300/600; `animation_delay` 300/600 | `group_slot` | | Animation needs `build/motion.js` |
| `sdc.haven_theme.section` | 18 | `width` 100–50%; `columns` 12 options; `views_columns` 2–6; `mobile_columns` 1–3; `margin_block_start` −48…128; `margin_block_end` 0…128; `padding_block_start/end` 0/16/32/64; `background_color` 5 roles; `background_media` obj; `restrict_width`, `section_header`, `section_footer` bool | `header_slot`, `main_slot`, `footer_slot` | | Three slots, not one |
| `sdc.haven_theme.text` | 12 | `text` rich; `text_size` text-xs…text-3xl (**7 steps**); `text_color` default/inverted/primary; `columns` 1–3 | — | | Design mentions only `normal`/`text-lg` |
| `sdc.haven_theme.card-icon` | 12 | `tile_size` square/4:3/16:9; `background_color`, `text_color`, `border_radius`, `icon`, `icon_size`, `icon_align`, `text_align`, `url`, `text`, `description` | — | | Icon names are Phosphor 2.1.1 |
| `sdc.haven_theme.button` | 10 | `variant` primary/secondary/accent/light; `size` small/medium/large; **`href` str (uri-reference)**; `outlined`, `icon_first`, `mobile_width`, `disabled` bool; `icon` 6 values; `label` | — | | **`href` exists**; 9 of 10 instances leave it empty |
| `sdc.haven_theme.badge` | 9 | `label`; `style` primary/secondary/accent; `icon` (default `rocket`); `icon_first` bool | — | | |
| `sdc.haven_theme.hero-billboard` | 6 | `height` full/large/ribbon; `flex_position` 4 values; `overlay_opacity` 0/20/40/**60/75**%; `object_position` top/center/bottom; `media` obj; `overlap_navbar`, `content_background` bool | `hero_slot` | | Design caps overlay at 40% |
| `sdc.haven_theme.hero-side-by-side` | 6 | `image_size` 2:1/16:9/3:2/4:3/1:1; `image_position` left/right; `background` primary/white/secondary/accent; `hero_flex_direction_mobile`; `padding` bool; `media` obj | `hero_slot` | | Ratios correctly quoted here |
| `sdc.haven_theme.card-logo` | 5 | `media` obj; `background_color`; `border_radius`; `url` | — | | |
| `sdc.haven_theme.card-testimonial` | 4 | `text`; `cite_name`, `cite_text`, `cite_url`; `style` default/white; `media` obj | — | | |
| `sdc.haven_theme.image` | 3 | `media` obj; **`size` enum broken (see upstream)**; `radius` small/large/extra-large; `caption`; `url` | — | | Missing from the design registry |
| `sdc.haven_theme.grid` | 2 | `width` 100–50%; `columns` 12 options; `mobile_columns` 1–3; `reverse_mobile` bool | `main_slot` | | |
| `sdc.haven_theme.icon` | 1 | `icon` str (required); `icon_size` extra-small…extra-large | — | | Missing from the design registry |
| `block.views_block.blog-all` | 1 | view `blog`, display `all` | — | | |
| `block.views_block.blog-latest` | 1 | view `blog`, display `latest` | — | | **Missing from the design registry** |
| `block.views_block.projects-all` | 1 | view `projects`, display `all` | — | | |
| `block.views_block.projects-featured` | 1 | view `projects`, display `featured` | — | | |

Declared but unused in seeded content: `accordion`, `accordion-container` (JS),
`anchor` (JS), `blockquote`, `card`, `card-feature`, `card-pricing`, `cta`,
`footer`, `navbar` (JS), `video` (JS).

---

## Decisions needed from the design or product owner

1. **Palette and fonts: re-skin or match Haven?** Haven ships oklch tokens with
   Geist and Gelasio (A2). `HANDOFF.md` specifies an entirely different hex
   palette with Young Serif and Atkinson Hyperlegible. Both are defensible — a
   re-skin shows Druxt theming independently of Drupal, a match shows parity —
   but the choice should be explicit, and the role list extended to cover
   Haven's `card`, `destructive`, `input` and `*-light`/`*-dark`.
2. **Breakpoint: 720px or 768px?** The design uses 720; Haven uses 768 almost
   everywhere (A3). Picking Haven's avoids a band where the two disagree.
3. **Headings up to 205 characters** (E). Cap the responsive size by length, or
   re-author the seeded content, or both.
4. **No focal point data** (C2). Either accept centre-crop, add a crop-exposing
   module, or derive crops in the frontend.
5. **Person portraits are 4:5, and there is one person.** The design's 1:1 circle
   crop will cut the single portrait (E).
6. **Donate has no backend at all** (C6). Confirm the simulated flow is
   acceptable and labelled as such.
7. **Newsletter and contact cannot be submitted** by a decoupled client (C5).
   Decide whether to add a submission module or simulate these too.
8. **Search has no endpoint** (C7). Decide whether to add one or cut search from
   the first release.
9. **Klaro config is unreadable** (C8). Decide whether to expose it or hard-code
   the cookie banner in the frontend.
10. **`top-tasks`, 7 links**, is the biggest menu and appears in no design (C4).
11. **Terms of service is a 31-character stub** (E).
12. **Alt text says "Test" on the Home hero** (E).

## Upstream issues worth filing

**Haven (`drupal/haven_theme`)**

1. **`image` component's `size` enum is corrupted by YAML.**
   `components/image/image.component.yml:19-24` declares `type: string` but
   leaves four values unquoted, so YAML parses them as sexagesimal integers:
   `16:9` → `969`, `3:2` → `182`, `4:3` → `243`, `1:1` → `61`. Only `"2:1"` is
   quoted. `hero-side-by-side.component.yml:44-49` quotes all five correctly,
   which shows the intent. The enum can never match a string value an editor
   picks. A one-line fix, and a clear bug.
2. **Sample content leaves `button.href` empty.** Nine of ten seeded buttons,
   including every "Donate now" and "Learn more", have no link (A4). The prop
   exists; the content does not use it.
3. **Sample content ships placeholder alt text**, including `alt="Test"` on the
   Home hero (E).

**Canvas (`drupal/canvas`)**

4. **No read-only access to the component registry.** `view` on `component`,
   `content_template`, `pattern` and `brand_kit` is gated behind
   `CanvasUiAccessCheck`, which only consults five `administer *` permissions
   (C11). A decoupled frontend that needs the prop schemas has to be granted
   write-capable permissions. A `view components` permission, or read-only
   exposure of enabled components, would fix it.
5. **`administer components` does not grant `view` on components.** The
   registry's own admin permission is not consulted by the access check, which
   is surprising enough to be worth reporting on its own (C11).

**Druxt (`drupal/druxt`, `druxt/druxt.js`)**

6. **No Canvas support.** Nothing in the module or the JS monorepo renders a
   component tree (D1, D3). At minimum the module could carry the read-only
   registry access from C11, since every decoupled Canvas site will need it.
