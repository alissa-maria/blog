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

I recently made my Dutch portfolio bilingual using Polylang (the free version) on a block theme. Translating the regular pages was straightforward. Translating the **front page** was not — WordPress fights you on it, and the clean solution is locked behind Polylang Pro at €100/year.

This post covers the workaround: how to serve a language-specific header, footer, and hero button on the front page using only CSS, no Pro licence and no PHP. There's a copy-paste fix at the top, and the reasoning (plus the mistakes worth avoiding) below it.

## The fix, up front

**The problem:** WordPress always renders your static homepage through the `Front Page` template, ignoring any per-page template you assign. So your English front page loads the Dutch header, and there's no built-in way to swap it in the free version.

**The workaround:** put *both* language headers (and footers, and hero buttons) into the single `Front Page` template, then show and hide them per language with CSS, keyed to the page ID.

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

Replace `383` with your English homepage's page ID. You'll find it in the `page-id-XXX` class that WordPress adds to the `<body>` tag — view source on your English homepage and look. That class is the key to the whole approach: it stays correct even when the front-page template overrides everything else.

If it doesn't work on the first try, jump to [Four things that will trip you up](#four-things-that-will-trip-you-up) — it's almost certainly one of those.

## What Polylang free handles fine

To be clear about where the limit actually is: Polylang's free version does the hard part well. It generates the `/en/` URLs, links each page to its translation, and makes the language switcher jump to the *matching* page rather than the homepage. For the regular pages, translating is genuinely just "duplicate, translate the text, done."

One thing is genuinely gated behind Pro: **automatically swapping a single Navigation block's menu by language.** In the Pro version, one nav block serves the right language on its own. In the free version it can't, and the string-translation panel can't reach a Navigation block either.

But that doesn't mean you can't translate the nav — it just means you do it manually, and it works cleanly. Build two Navigation blocks (one per language), put each in its own header template part, and let a language-specific template serve the right header. Same approach for the footer. On every regular page, that gives you a fully translated header and footer with no Pro licence.

The catch is the front page, which ignores template assignments. That's the one case the manual approach can't reach on its own — and it's what the rest of this post solves.

## Why the front page is different

Every other page obeys a simple rule: assign it a template, it uses that template. So the standard bilingual setup works fine for them:

1. Duplicate your header template part → `header-EN`, and swap in the English navigation.
2. Do the same for the footer.
3. Create an English template (duplicate `Index`) that uses `header-EN` and `footer-EN`.
4. Assign your English pages to it.

The front page ignores step 4. WordPress has a fixed template hierarchy, and whatever page is set as the static homepage in **Settings → Reading** always renders through the `Front Page` template — no matter what you choose in that page's template dropdown. Its description even says it "takes precedence over all templates," which turns out to be literal.

The result: your English homepage renders the Dutch header. A visitor clicks *English*, the URL changes to `/en/`, but the page still looks Dutch. If they click a nav link, they're back on Dutch pages. It's more confusing than leaving it untranslated.

## The workaround, step by step

Since you can't make WordPress *pick* the right template, you put both options *into* the template it insists on using, and hide the wrong one.

**Find your hook.** CSS needs something in the HTML that differs by language. Polylang doesn't add a language class to `<body>` by default, but WordPress always adds `page-id-XXX`, and — importantly — that class survives the front-page override. Confirm it by viewing source on both homepages:

```html
<!-- Dutch homepage -->
<body class="home ... page page-id-77 ...">

<!-- English homepage (/en/) -->
<body class="home ... page page-id-383 ...">
```

**Build it.**

1. In the `Front Page` template, add both header template parts and both footer parts. Leave them all visible (see gotcha #1).
2. Give each one a class under **Advanced → Additional CSS class(es)**: `header-nl`, `header-en`, `footer-nl`, `footer-en`.
3. Add the CSS from the top of this post, using your English homepage's page ID.

That's it. The Dutch homepage shows the Dutch header; the English homepage matches `body.page-id-383` and swaps to the English one. Pure CSS, entered in the theme's additional-CSS box.

It's worth being honest about the trade-off: this loads both headers into the DOM on every front-page request and hides one. For a personal site that's a negligible amount of extra markup. For a high-traffic site, the proper fix is a small `functions.php` snippet or Pro — but for most portfolios, this is fine and invisible.

## Four things that will trip you up

These are the details that cost me the most time, and that most tutorials leave out.

**1. The editor's "hide block" toggle is not the same as `display: none`.** Hiding a block in the editor can stop it rendering on the front end entirely, so your CSS has nothing to reveal. Leave every block *visible* in the editor and let the CSS handle all hiding. Once it's all set up, the block editor even renders the correct version for you, so there's no confusion.

**2. Watch your specificity.** A bare `.header-en { display: none; }` may lose to the theme's own layout rules on `.wp-block-template-part`. Target the full selector — `.wp-block-template-part.header-en` — so you have enough weight to win. Try this before reaching for `!important`; in my case the specificity bump made `!important` unnecessary.

**3. Put both hero buttons in one Buttons block.** If your cover image has a button, it needs translating too. Duplicate it (`btn-nl`, `btn-en`, same page-id swap), but keep both buttons inside a *single* Buttons block rather than two stacked ones. Two separate Buttons blocks leave an empty wrapper behind when one is hidden, which throws off the vertical spacing. One wrapper keeps the position predictable.

**4. Check for a stray `margin-top`.** After everything worked, my English header still sat 24px lower than the Dutch one. The cause wasn't the layout — the English header instance had a `margin-top` the Dutch one didn't, most likely left over from duplicating the template part. A quick `margin-top: 0` on that instance fixed it. Duplication in the block editor doesn't always carry spacing values consistently, so it's worth checking margins if two "identical" parts don't line up.

## Worth it?

The end result is a fully bilingual portfolio, front page included, for the cost of a domain name. The only remaining imperfection is invisible unless you load both language homepages side by side — which no real visitor does.

If you're translating a small site and staring down €100 to swap one header, this should save you an evening. The fix isn't elegant, but it's free, it's entirely in the WordPress GUI, and it works.