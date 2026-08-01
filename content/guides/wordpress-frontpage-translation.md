---
title: "Translating a WordPress front page without Polylang Pro"
description: "How an afternoon of stubbornness saved my small WordPress site €100 a year — translating a front page without Polylang Pro."
date: 2026-07-30
tags:
  - wordpress
  - polylang
  - css
  - block-themes
---

I recently made my Dutch portfolio bilingual using Polylang (the free version) on a block theme. The free version does the hard part well: it generates the `/en/` URLs, links each page to its translation, and makes the switcher jump to the *matching* page rather than the homepage. Translating a regular page is just "duplicate, translate the text, done."

The front page is the exception. WordPress renders your static homepage through the `Front Page` template no matter what you assign it, so the English front page loads the Dutch header — and the clean fix sits behind Polylang Pro at €100/year. Here's the workaround: a language-specific header, footer, and whatever else your front page holds, using only CSS.

This assumes a block theme, a static front page, and Polylang Free. Copy-paste fix first, reasoning and mistakes below.

## The fix, up front

Put *both* language versions of every block that carries text into the single `Front Page` template, then show and hide them per language with CSS, keyed to the page ID.

Throughout, `nl` is the language the site started in and `en` the one added later — substitute your own. The direction is what matters: you hide the *added* language by default and reveal it on its own homepage.

```css
/* English header/footer hidden everywhere by default */
.wp-block-template-part.header-en,
.wp-block-template-part.footer-en { display: none; }

/* On the English homepage only: hide Dutch, show English */
body.page-id-383 .wp-block-template-part.header-nl,
body.page-id-383 .wp-block-template-part.footer-nl { display: none; }

body.page-id-383 .wp-block-template-part.header-en,
body.page-id-383 .wp-block-template-part.footer-en { display: block; }
```

Replace `383` with your English homepage's page ID, taken from the `page-id-XXX` class WordPress puts on the `<body>` tag. That class is the whole trick: even though both homepages render through the same `Front Page` template, WordPress still tells you which page it queried. CSS can key off that page ID, so the template behaves differently for `/` and `/en/`.

The example only covers the header and footer. Any other translated block follows exactly the same pattern — class the block, hide the added-language version by default, reveal it under the page ID. Mine was a button on the hero image; yours might be a tagline, or nothing at all. More than two languages works the same way, with proportionally more of it.

If it doesn't work on the first try, jump to [Four things that will trip you up](#four-things-that-will-trip-you-up) — it's almost certainly one of those.

## Why the front page needs a workaround at all

The problem starts with navigation. One thing is properly gated behind Pro: **swapping a single Navigation block's menu by language**. In the free version, one Navigation block can't do it, and the string-translation panel can't reach a Navigation block either. You can still translate the nav, you just do it manually — and on regular pages that works cleanly:

1. Duplicate your header template part → `header-EN`, and swap in the English navigation.
2. Do the same for the footer.
3. Duplicate whichever template your pages render through — usually `Pages` or `Index` — and point it at `header-EN` and `footer-EN`.
4. Assign your English pages to it.

The front page ignores step 4. Whatever page is set as the static homepage in **Settings → Reading** always renders through `Front Page`, regardless of that page's template dropdown. The template's own description says it "takes precedence over all templates," which turns out to be literal.

So a visitor clicks *English*, the URL changes to `/en/`, and the page still looks Dutch — and one nav click puts them back on Dutch pages. More confusing than leaving it untranslated.

Since you can't make WordPress *pick* the right template, you put both options into the one it insists on using and hide the wrong one.

**Find your hook.** CSS needs something in the HTML that differs by language. Polylang adds no language class to `<body>`, but WordPress always adds `page-id-XXX`, and that survives the override. View source on both homepages:

```html
<!-- Dutch homepage -->
<body class="home ... page page-id-77 ...">

<!-- English homepage (/en/) -->
<body class="home ... page page-id-383 ...">
```

**Build it.**

1. In the `Front Page` template, add both header template parts and both footer parts. Leave them all visible (see gotcha #1).
2. Give each a class under **Advanced → Additional CSS class(es)**: `header-nl`, `header-en`, `footer-nl`, `footer-en`.
3. Do the same for any other block with text in it — for me a button on the cover image, as `btn-nl` and `btn-en` (see gotcha #3 for where to put them).
4. Add the CSS from the top of this post, using your own page ID.

The trade-off: both headers load into the DOM on every front-page request and one gets hidden. Negligible for a portfolio. For a high-traffic site the proper fix is a small `functions.php` snippet, or Pro.

## Four things that will trip you up

The details that cost me the most time, and that most tutorials leave out.

**1. The editor's "hide block" toggle is not the same as `display: none`.** Hiding a block in the editor can stop it rendering on the front end entirely, leaving your CSS nothing to reveal. Leave every block *visible* and let the CSS do all the hiding — once it's set up, the editor renders the correct version anyway.

**2. Watch your specificity.** A bare `.header-en { display: none; }` may lose to the theme's own rules on `.wp-block-template-part`. Target the full selector instead: `.wp-block-template-part.header-en`. Try that before reaching for `!important`; the specificity bump made it unnecessary for me.

**3. Hiding a block leaves its wrapper behind.** My translated hero button started out as two stacked Buttons blocks, and hiding one left an empty wrapper still taking up space. Putting both buttons inside a *single* Buttons block fixed it. Generally: if hiding one of two siblings shifts the layout, the leftover wrapper is why.

**4. Check for a stray `margin-top`.** After everything worked, my English header still sat 24px lower than the Dutch one — a `margin-top` on that instance, left over from duplicating the template part. `margin-top: 0` fixed it. Duplication doesn't always carry spacing values consistently, so check the margins when two "identical" parts don't line up.

## Worth it?

Very. A fully bilingual portfolio, front page included, for the cost of a domain name — and I was more pleased with it than the size of the problem really warrants.

What makes it hold up isn't the CSS, which is four rules. It's that the duplication lives in the template rather than in anything I have to maintain: each header is a template part, so editing the Dutch nav updates it everywhere it appears. Paste blocks around instead and you'll be editing the same menu in four places by December.

Not the cleanest solution available, but cleaner than "hack" suggests — and clean is mostly a decision you make while building. If you're staring down €100 to swap one header, this should save you an evening.