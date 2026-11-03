---
name: saas-sidebar-redesign
description: Redesign a Vue 3 app's UI into a modern SaaS-style interface with a vertical left sidebar instead of a top nav bar, a consistent spacing scale, and a polished professional look. Use this skill whenever the user wants to move navigation from the top to the side, add a sidebar or app shell, modernize or "SaaS-ify" a Vue dashboard, make the UI look more professional or consistent, or standardize spacing, cards and typography, even if they don't say "sidebar" explicitly.
---

# SaaS Sidebar Redesign for Vue 3

Turn a Vue 3 + vue-router app that has a top navigation bar into a modern SaaS application shell: a persistent left sidebar, a slim content header, and a consistent visual system. The goal is a change in structure and polish, not in behavior. Routes, data loading, stores and business logic stay as they are.

## Before you start

1. **Read the existing shell.** Open `App.vue`, `main.js` and the router config. Find the top nav, every `router-link`, and anything else sitting beside it (language switcher, profile menu, global filter bar). These need a new home in the new layout, so list them before changing anything.
2. **Find the existing design tokens.** Check for CSS variables, a theme file, or the palette used in `App.vue`. Reuse the app's colors and fonts. A redesign that keeps the brand feels like polish, while one that swaps the palette feels like a different product. If the project has a design system in its `CLAUDE.md`, treat it as binding (for example, "no emojis in UI" means icons must be SVG, not emoji).
3. **Check project rules for who edits what.** Some repos require `.vue` files to be created or significantly changed by a specific subagent. In this repo, `CLAUDE.md` says to delegate any significant `.vue` change to the `vue-expert` subagent. Follow that, and pass it this skill's guidance plus the inventory you built in step 1. If you are running as a subagent yourself and can't delegate, make the edits directly and say so in your report.
4. **Work on a branch or worktree.** A redesign touches many files. Make sure it is easy to review and easy to discard.

## Target layout

```
+-----------+--------------------------------------------+
| Sidebar   |  Content header (title, filters, tools)    |
| (brand)   +--------------------------------------------+
| nav items |                                            |
|           |  <router-view />  (page content)           |
| (footer:  |                                            |
|  profile) |                                            |
+-----------+--------------------------------------------+
```

- **Sidebar** is fixed to the left and full height. Brand at the top, navigation in the middle, and account or secondary items (profile menu, language switcher) pinned at the bottom.
- **Content area** scrolls independently of the sidebar, and has a slim header for global controls such as filters. Show a page title in that header only if pages don't already render their own heading. Check first, because duplicated titles look worse than none.
- A fixed-height shell (`100vh` with an inner scroller) moves scrolling off `window`. Check anything that depended on window scroll: sticky table headers, scroll listeners, "scroll to top" on route change.
- Put the shell in `App.vue`, or extract `AppSidebar.vue` when the sidebar logic outgrows it (rule of thumb: over about 100 lines of template). Keep pages unaware of the sidebar.

## Sidebar behavior

Build these in by default, because users expect them from a SaaS shell:

- **Active state from the router.** Use `router-link`'s built-in active classes rather than hand-comparing `$route.path` per link, which gets verbose and breaks for nested routes. The root route needs care: `/` matches every path under `active-class`, so give it `exact-active-class` only. Map both to one class:
  ```vue
  <router-link :to="item.to" active-class="" exact-active-class="is-active" v-if="item.to === '/'" />
  <router-link :to="item.to" active-class="is-active" exact-active-class="is-active" v-else />
  ```
  Drive the links from an array of `{ to, labelKey, icon }` so adding a page is one line.
- **Collapsible to an icon rail.** A toggle switches between full width (about 240px) and a narrow rail (about 64px) showing only icons. Persist the choice in `localStorage` so it survives reloads. When collapsed, expose each label through `title` or `aria-label` so icons stay understandable.
- **Mobile drawer.** Below a breakpoint (about 768px) the sidebar leaves the layout and becomes an overlay drawer opened by a menu button in the content header, with a backdrop that closes it. Close it on route change, otherwise users navigate and still see the menu covering the page. The drawer should ignore the collapsed state and always show full labels. Wrap the collapsed-rail CSS in `@media (min-width: 768px)` so no JS breakpoint is needed.
- **Embedded components.** Components that were built for a top bar (language switcher, profile menu) usually open dropdowns downward and right-aligned, and don't hide their labels. In the sidebar footer, restyle them from the parent with a higher-specificity selector (scoped styles block plain overrides, so use `:deep()` or a parent-class prefix), open menus upward, and hide text labels in the rail. Prefer this over editing each component unless the change is large.
- **Accessibility.** Wrap links in `<nav aria-label="Main">`, make the toggle a real `<button>` with `aria-expanded`, keep visible focus rings, and let Escape close the drawer.

Use the app's existing i18n for labels if it has any. Never hard-code strings into a project that is already translated. New UI text (collapse and menu aria-labels) needs new keys in every locale file, because many `t()` helpers fall back silently to the key name, which hides a missing translation. If you find labels that were already hard-coded, leave them and mention them in your report rather than widening the change.

## Visual system

Consistency is most of what makes an interface look professional, so define the system once as CSS variables on `:root` and use them everywhere instead of ad hoc values.

- **Spacing scale:** 4px base, using only 4, 8, 12, 16, 24, 32, 48. Page padding is 24-32px, gaps between cards are 16-24px, and card padding is 16-24px. Replace stray values like 13px or 22px with the nearest scale step.
- **Radius and elevation:** one card radius (8-12px), one control radius (6-8px), and one subtle shadow plus 1px border for cards. Avoid stacking heavy shadows.
- **Typography:** one font stack (system UI is fine). Page title around 20-24px semibold, section titles 16px, body 14px, secondary text 12-13px in the muted color. Keep line-height around 1.5.
- **Color:** a neutral surface and border set, one accent color for active and primary states, and status colors only for status. Check that muted text still meets contrast on its background.
- **Components:** cards, buttons, inputs, tables and badges should look like one family. Promote repeated styles into shared classes or small components in `App.vue` or a shared stylesheet, so pages stop each carrying their own copy.
- **Icons:** inline SVG with a consistent stroke width, sized 18-20px. Respect any "no emoji" rule.

## Process

1. Write the inventory (nav items, extras, tokens already in use).
2. Add the design tokens as CSS variables.
3. Build the shell: sidebar, content header, mobile drawer, collapse toggle. Move the top-nav items, switcher, profile menu and filter bar to their new places. Delete the old top-nav markup and CSS rather than leaving it hidden, since dead styles confuse the next reader. Then search for leftovers that depended on the old layout: selectors like `.nav-container > .x`, `position: sticky; top: <nav height>` on filter bars, and `min-height`/padding that compensated for the old header.
4. Apply the tokens to shared styles (global classes in `App.vue`, shared components). Pages often carry their own scoped copies of spacing and font sizes. Don't rewrite every view. Stop at shared styles, and list the pages with remaining one-off values (and any lacking mobile media queries) in your report so the user can choose follow-up work. Do not change page logic.
5. Verify in the real app (see below), then summarize what changed.

Explain non-obvious choices in short code comments (for example why the drawer closes on route change), because future readers can't see the reasoning in the CSS alone.

## Verification

- Start the dev server and load every route. Nothing should be hidden behind the sidebar, and the page should not scroll horizontally.
- Test three widths: desktop (about 1440px), tablet (about 900px) and mobile (about 390px).
- Toggle collapse, reload to confirm it persists, then open and close the mobile drawer, and navigate with it open.
- Check the browser console for errors and warnings.
- If browser automation is available (the project may provide Playwright MCP tools), take screenshots at each width and click through the collapse toggle and drawer.
- If only one-shot headless screenshots are possible, they can't click or reload, so say plainly that collapse, persistence and drawer behavior were not exercised. Headless Chromium/Edge also enforces a minimum window width of about 500px, so a 390px screenshot is cropped and can't be trusted for overflow. Don't report mobile as verified from it. Ask the user to check those parts in their browser.
- Always finish with a "verified / not verified" list, because a build that passes says nothing about how the layout looks.

## Pitfalls

- Don't make the content area `width: 100vw` next to a sidebar. It causes horizontal overflow. Use flex or grid, with `min-width: 0` on the content column so wide tables and charts don't push the layout.
- Fixed-position modals and dropdowns inside pages can end up under or over the sidebar. Check their `z-index` against the sidebar and the drawer backdrop.
- Charts sized from their container may need a resize or re-render when the sidebar collapses, so check them after toggling.
- Don't rename routes or change route paths. Bookmarks and tests depend on them.
- Keep the change reviewable: shell and tokens first, page sweeps after, and no unrelated refactors.
