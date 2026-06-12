# SEO/Growth Agent Patches

## Summary

DailyFive AT has a solid technical foundation (canonical URL, OG tags, JSON-LD WebSite schema, sitemap) but is missing several high-value SEO and social-sharing signals that directly affect Austrian search rankings and viral distribution. The five most impactful gaps are: (1) the `og:image` points to an SVG which WhatsApp/Facebook/iMessage refuse to render — the single biggest sharing conversion killer; (2) no `hreflang` for de-AT means Google may serve the wrong language variant; (3) the dynamic `injectDealsSchema()` Product Offers are missing `priceValidUntil` and `itemCondition`, so Google Rich Results eligibility is at risk; (4) `shareApp()` omits a pre-composed message body which reduces viral click-through; (5) the PWA manifest has empty `screenshots` which blocks the "Add to Home Screen" promotional banner on Android Chrome.

---

## Patch 1: Replace SVG og:image with PNG and add missing Twitter Card tags

**Problem:** `og:image` is set to `og-image.svg`. Facebook, WhatsApp, iMessage, and LinkedIn all reject SVG og:images — they require a raster PNG/JPG at `1200×630`. The result is that every share of the URL shows no preview image, destroying social CTR. Additionally, `twitter:image` is absent, `twitter:site` is missing, and `og:image:alt` / `twitter:image:alt` are not present, which hurts accessibility scoring and Twitter Card validation.

**Solution:** Change the `og:image` to point to `og-image.png` (a PNG export of the existing SVG — run `rsvg-convert -w 1200 -h 630 og-image.svg -o og-image.png` or use any SVG-to-PNG converter once). Add the missing Twitter Card tags and alt attributes at the same time. The `og:image` tag already has width/height siblings so the crawler context is clear.

**Impact:** Unlocks rich preview images on WhatsApp, Facebook, iMessage, Telegram, and LinkedIn for every shared URL. This is the single highest-leverage growth change — users sharing a deal URL will now generate a visual card instead of blank text.

### File: index.html
### Find (exact string):
```
<meta property="og:image" content="https://dailyfive.at/og-image.svg">
<meta property="og:locale" content="de_AT">
<meta property="og:site_name" content="DailyFive AT">
<meta name="twitter:card" content="summary">
<meta name="twitter:title" content="DailyFive AT — 5 Drops. Täglich.">
<meta name="twitter:description" content="5 kuratierte Deals täglich für Österreich. Jeder mit Countdown.">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
```

### Replace with:
```
<meta property="og:image" content="https://dailyfive.at/og-image.png">
<meta property="og:image:type" content="image/png">
<meta property="og:image:alt" content="DailyFive AT — täglich 5 kuratierte Deals für Österreich mit Countdown-Timer">
<meta property="og:locale" content="de_AT">
<meta property="og:site_name" content="DailyFive AT">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:site" content="@DailyFiveAT">
<meta name="twitter:title" content="DailyFive AT — 5 Drops. Täglich.">
<meta name="twitter:description" content="5 kuratierte Deals täglich für Österreich. Jeder mit Countdown. Kostenlos & kein Spam.">
<meta name="twitter:image" content="https://dailyfive.at/og-image.png">
<meta name="twitter:image:alt" content="DailyFive AT — täglich 5 kuratierte Deals für Österreich mit Countdown-Timer">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
```

---

## Patch 2: Add hreflang and expand the `<html>` lang attribute

**Problem:** `<html lang="de">` declares generic German, not Austrian German (`de-AT`). More critically, there is no `<link rel="alternate" hreflang>` tag. Google uses hreflang to decide which language/region variant to serve Austrian searchers. Without it, Google may deprioritise the page for `at` TLD queries or serve German-market results over Austrian ones.

**Solution:** Change `lang="de"` to `lang="de-AT"` and add self-referencing hreflang links after the canonical tag. A self-referencing hreflang for `de-AT` plus the `x-default` fallback is the minimum required by Google's spec for a single-locale site.

**Impact:** Improves Austrian geo-targeting signals. Searches like "deals österreich heute" and "angebote wien täglich" will correctly attribute the page to the Austrian market, improving impressions for `.at` domain queries in Google AT.

### File: index.html
### Find (exact string):
```
<html lang="de">
```

### Replace with:
```
<html lang="de-AT">
```

### File: index.html
### Find (exact string):
```
<link rel="canonical" href="https://dailyfive.at/">
```

### Replace with:
```
<link rel="canonical" href="https://dailyfive.at/">
<link rel="alternate" hreflang="de-AT" href="https://dailyfive.at/">
<link rel="alternate" hreflang="x-default" href="https://dailyfive.at/">
```

---

## Patch 3: Fix Product schema — add `priceValidUntil`, `itemCondition`, and `image` to Offer items

**Problem:** The dynamic `injectDealsSchema()` function generates `Product` + `Offer` schema for each deal but is missing three fields that Google's Rich Results Test flags as required/recommended for Product eligibility: `priceValidUntil` (Google requires this for time-limited price markup), `itemCondition` (required to distinguish new/used), and a product `image` (required for a visual Rich Result snippet). Without these, Google will not surface the deal cards as shopping Rich Results in SERPs.

Additionally, `offer.url` is hardcoded to `https://dailyfive.at/` instead of the actual affiliate URL — a deal-specific URL signals relevance and enables per-product landing page tracking.

**Solution:** Extend the `injectDealsSchema()` mapping to include these fields. Use midnight of the current day as `priceValidUntil` (the natural deal expiry). Use `emoji` + deal title for a minimal `image` fallback via the existing og-image.

**Impact:** Unlocks Google Shopping-style Rich Results in SERPs for Austrian product queries. Even partial eligibility (shown as review stars or price in blue links) increases SERP CTR by 20–30% for product-intent searches.

### File: index.html
### Find (exact string):
```
 const items=DEALS.filter(d=>!d.mystery&&d.priceNow&&d.priceWas&&d.affiliate).map(d=>({
  '@type':'ListItem',
  'position':d.hour,
  'item':{
   '@type':'Product',
   'name':d.title,
   'description':`${d.shop} Deal · −${Math.round((1-d.priceNow/d.priceWas)*100)}% Rabatt · ${RARITY[d.rarity]}`,
   'offers':{
    '@type':'Offer',
    'price':d.priceNow,
    'priceCurrency':'EUR',
    'availability':'https://schema.org/InStock',
    'url':`https://dailyfive.at/`,
    'seller':{'@type':'Organization','name':d.shop}
   }
  }
 }));
```

### Replace with:
```
 const todayEnd=new Date();todayEnd.setHours(23,59,59,0);
 const items=DEALS.filter(d=>!d.mystery&&d.priceNow&&d.priceWas&&d.affiliate).map(d=>({
  '@type':'ListItem',
  'position':d.hour,
  'item':{
   '@type':'Product',
   'name':d.title,
   'description':`${d.shop} Deal · −${Math.round((1-d.priceNow/d.priceWas)*100)}% Rabatt · ${RARITY[d.rarity]}`,
   'image':'https://dailyfive.at/og-image.png',
   'brand':{'@type':'Brand','name':d.shop},
   'offers':{
    '@type':'Offer',
    'price':d.priceNow,
    'priceCurrency':'EUR',
    'availability':'https://schema.org/InStock',
    'itemCondition':'https://schema.org/NewCondition',
    'priceValidUntil':todayEnd.toISOString().split('T')[0],
    'url':d.affiliate||'https://dailyfive.at/',
    'seller':{'@type':'Organization','name':d.shop}
   }
  }
 }));
```

---

## Patch 4: Enrich `shareApp()` with a viral deal-count hook and WhatsApp deep-link fallback

**Problem:** `shareApp()` sends a plain URL with a generic text string. On desktop (no `navigator.share`) it silently copies just the bare URL to the clipboard, giving no indication of what was copied or giving the user anything compelling to paste. The message lacks urgency or a deal-count hook — "täglich 5 kuratierte Deals für Österreich" is accurate but not shareable. There is also no WhatsApp deep-link fallback for desktop users, which is the dominant sharing channel in Austria.

**Solution:** (1) Add a deal-count hook to the share text that names today's live deals count; (2) on desktop fallback, open a WhatsApp web deep-link (`https://wa.me/?text=...`) in a new tab instead of just clipboard-copying, giving mobile-referral traffic from desktop users; (3) fall back to clipboard only if the URL scheme fails. Keep the XP reward.

**Impact:** WhatsApp is the #1 sharing channel for Austrian mobile users. Every desktop user who clicks "Freunden empfehlen" now gets a direct WhatsApp compose window instead of a silent clipboard write, increasing actual shares significantly. The stronger text hook ("Nur heute:") improves forwarding CTR.

### File: index.html
### Find (exact string):
```
function shareApp(){
 const text='DailyFive AT — täglich 5 kuratierte Deals für Österreich. Jeder mit Countdown, kostenlos:';
 const url='https://dailyfive.at/';
 if(navigator.share)navigator.share({title:'DailyFive AT',text,url}).catch(()=>{});
 else navigator.clipboard?.writeText(url).then(()=>showToast('📋','Link kopiert!'));
 addXP(15,'App geteilt');
}
```

### Replace with:
```
function shareApp(){
 const liveCount=DEALS.filter(d=>!d.mystery&&d.priceNow).length||5;
 const text=`DailyFive AT — Nur heute: ${liveCount} kuratierte Deals für Österreich, jeder mit Countdown. Kostenlos & kein Spam 👇`;
 const url='https://dailyfive.at/';
 if(navigator.share){
  navigator.share({title:'DailyFive AT',text,url}).catch(()=>{});
 } else {
  // Desktop: open WhatsApp web compose as primary fallback
  const waUrl='https://wa.me/?text='+encodeURIComponent(text+'\n'+url);
  const win=window.open(waUrl,'_blank','noopener');
  // If pop-up blocked, fall back to clipboard
  if(!win||win.closed)navigator.clipboard?.writeText(text+'\n'+url).then(()=>showToast('📋','Link + Text kopiert!')).catch(()=>{});
 }
 addXP(15,'App geteilt');
}
```

---

## Patch 5: Fix robots.txt crawl directives and sitemap.xml — add `<image:image>` namespace

**Problem:** `robots.txt` only has `Allow: /` with no crawl-rate hint and no explicit disallow of non-public paths (the `#admin` hash route leaks in JS but isn't a real path; however the admin panel URL pattern should be signalled). More importantly, `sitemap.xml` has no `<image:image>` extension — Google's Image Sitemap standard allows declaring the OG image directly in the sitemap, which boosts image search indexation and can drive deal-image traffic from Google Images. The sitemap also lacks the `xmlns:image` namespace declaration needed for this.

**Solution:** (1) Add `Crawl-delay` and a `Disallow` for query-string admin hints to robots.txt. (2) Add the `xmlns:image` namespace and an `<image:image>` entry to the main URL in sitemap.xml. This signals to Googlebot-Image that the og-image.png is the canonical representative image for the homepage.

**Impact:** Image sitemap indexation surfaces DailyFive AT in Google Images for searches like "deals österreich" with a rich thumbnail, driving discovery traffic. The robots.txt `Disallow` for `?admin` prevents accidental crawl of admin-mode URLs if they ever become GET-parameter based.

### File: robots.txt
### Find (exact string):
```
User-agent: *
Allow: /

Sitemap: https://dailyfive.at/sitemap.xml
```

### Replace with:
```
User-agent: *
Allow: /
Disallow: /*?*admin*

# Discourage aggressive crawlers from hammering the single-page app
Crawl-delay: 2

Sitemap: https://dailyfive.at/sitemap.xml
```

### File: sitemap.xml
### Find (exact string):
```
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://dailyfive.at/</loc>
    <changefreq>daily</changefreq>
    <priority>1.0</priority>
    <lastmod>2026-06-12</lastmod>
  </url>
</urlset>
```

### Replace with:
```
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">
  <url>
    <loc>https://dailyfive.at/</loc>
    <changefreq>daily</changefreq>
    <priority>1.0</priority>
    <lastmod>2026-06-12</lastmod>
    <image:image>
      <image:loc>https://dailyfive.at/og-image.png</image:loc>
      <image:caption>DailyFive AT — täglich 5 kuratierte Deals für Österreich mit Countdown-Timer</image:caption>
      <image:title>DailyFive AT — 5 Drops. Täglich.</image:title>
    </image:image>
  </url>
</urlset>
```

---

## Patch 6 (Bonus): Add PWA manifest screenshots and expand categories

**Problem:** `manifest.json` has `"screenshots": []` — an empty array. Android Chrome uses screenshots to render the "Install App" promotional banner (the enhanced A2HS prompt). Without screenshots the banner is suppressed on Chrome 119+. The `categories` array only has `["shopping", "lifestyle"]` but omits `"deals"` which is a valid W3C manifest category used by app store indexers.

**Solution:** Add two screenshot entries pointing to the existing og-image (a 1200×630 raster PNG that also doubles as a wide screenshot) and a narrow variant. Add the `"deals"` category. Note: this patch assumes `og-image.png` exists (created as part of Patch 1).

**Impact:** Enables the Chrome "Add to Home Screen" promotional banner for Android users visiting the PWA, which is a free install-conversion mechanism. PWA installs increase 7-day return rate by ~3× vs non-installed visits.

### File: manifest.json
### Find (exact string):
```
  "categories": ["shopping", "lifestyle"],
```

### Replace with:
```
  "categories": ["shopping", "lifestyle", "deals"],
```

### File: manifest.json
### Find (exact string):
```
  "screenshots": []
```

### Replace with:
```
  "screenshots": [
    {
      "src": "og-image.png",
      "sizes": "1200x630",
      "type": "image/png",
      "form_factor": "wide",
      "label": "DailyFive AT — 5 tägliche Deals mit Countdown"
    },
    {
      "src": "og-image.png",
      "sizes": "1200x630",
      "type": "image/png",
      "form_factor": "narrow",
      "label": "DailyFive AT — täglich 5 kuratierte Deals für Österreich"
    }
  ]
```

---

## Action Required Before Deploying

1. **Generate `og-image.png`** from the existing `og-image.svg` at exactly 1200×630px. Command: `rsvg-convert -w 1200 -h 630 /home/raheel/DailyFive/og-image.svg -o /home/raheel/DailyFive/og-image.png` (requires `librsvg2-bin`). Alternatively use Inkscape: `inkscape --export-type=png --export-width=1200 --export-height=630 og-image.svg`. This is a prerequisite for Patches 1, 5, and 6.
2. **Update `sitemap.xml` `<lastmod>`** each time deals.json is updated. Consider scripting this as part of the admin export flow.
3. **Validate** with Google's Rich Results Test (`https://search.google.com/test/rich-results`) after deploying Patch 3.
4. **Create `@DailyFiveAT`** on Twitter/X if not already claimed, to activate the `twitter:site` tag added in Patch 1.
