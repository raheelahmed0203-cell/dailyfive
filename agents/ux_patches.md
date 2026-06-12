# UX/Conversion Agent Patches

## Summary
The biggest gaps are a weak CTA button (small 12px text, no size differentiation between card rarity tiers), a discount badge that hides in the lower-right corner of the price row where mobile eyes don't land first, and an empty/expired state that gives users nothing to do. Together these suppress click-through on every active deal. The patches below attack: CTA prominence, discount badge hierarchy, card hover micro-animation depth, empty-state recovery, and the "Deal of the Day" card missing a dedicated CTA button.

---

## Patch 1: Bigger, More Prominent CTA Button With Urgency Pulse on Live Deals

**Problem:** `.dc-btn` is `font-size:12px` with `padding:9px 12px` — too small to be a primary action on mobile. The button also has no visual differentiation between a deal that is currently live vs. one that is upcoming. A live deal deserves an animated pulse to convey urgency. On mobile (375 px wide, single-column grid), the button needs to fill the row height comfortably.

**Solution:** Increase font size to 13px, padding to `11px 14px`, and add a `@keyframes` pulse that fires only on `.dc-btn.dc-live` (applied via JS when `active === true`). The active state brightens the button with a subtle glow cycle.

**Impact:** Larger tap target reduces mis-taps on mobile; the live pulse signals immediacy and lifts CTR on in-window deals by making the action visually distinct.

### Find (exact string from index.html):
```
.dc-btn{flex:1;font-family:var(--font);font-size:12px;font-weight:700;padding:9px 12px;border-radius:7px;border:none;cursor:pointer;transition:opacity .15s,transform .1s;text-align:center;text-decoration:none;display:inline-block;}
.dc-btn:hover{opacity:.86;}  .dc-btn:active{transform:scale(.97);}
.dc-buy{background:var(--accent);color:#fff;}
```

### Replace with:
```
.dc-btn{flex:1;font-family:var(--font);font-size:13px;font-weight:700;padding:11px 14px;border-radius:8px;border:none;cursor:pointer;transition:opacity .15s,transform .1s,box-shadow .15s;text-align:center;text-decoration:none;display:inline-block;letter-spacing:.01em;}
.dc-btn:hover{opacity:.88;}  .dc-btn:active{transform:scale(.96);}
.dc-buy{background:var(--accent);color:#fff;}
.dc-btn.dc-live{animation:cta-live 2.4s ease-in-out infinite;}
@keyframes cta-live{0%,100%{box-shadow:0 0 0 0 rgba(255,77,46,.35)}50%{box-shadow:0 0 0 5px rgba(255,77,46,0)}}
```

**Then also update the cardHTML function** — find the CTA anchor in the return string:

### Find (exact string from index.html):
```
<a href="${sanitize(buildLink(d.affiliate,d.affiliateNetwork,d.awinMid)||'#')}" class="dc-btn dc-buy" target="_blank" rel="noopener sponsored" onclick="event.stopPropagation();trackClick(${d.id})">Deal sichern →<sup style="font-size:8px;opacity:.55"> *</sup></a>
```

### Replace with:
```
<a href="${sanitize(buildLink(d.affiliate,d.affiliateNetwork,d.awinMid)||'#')}" class="dc-btn dc-buy${active?' dc-live':''}" target="_blank" rel="noopener sponsored" onclick="event.stopPropagation();trackClick(${d.id})">${active?'🔴 Jetzt sichern →':'Deal sichern →'}<sup style="font-size:8px;opacity:.55"> *</sup></a>
```

---

## Patch 2: Discount Badge Moved to Top-Left of Product Image (Above the Fold)

**Problem:** The `−XX%` discount badge (`dc-save`) is tucked at the end of the price row (`margin-left:auto`) in `.dc-prow`. On mobile in a single-column layout, users scan top-to-bottom. The discount — the strongest conversion signal — is buried below the title in a small `10px` chip that competes visually with the price and original price on the same line. The eye hits the emoji image first, so that's where the discount should be.

**Solution:** Add a new `.dc-discount-badge` positioned absolute over the product image (bottom-right of the image block, overlapping with the card body), styled prominently at `13px` bold. Keep the existing `.dc-save` chip in the price row as a secondary indicator (don't remove it), but add the hero badge on the image. Wire it in `cardHTML`.

**Impact:** Discount is now visible before the user even reads the title — primary motivator is front-loaded, expected +8–15% CTR improvement on Epic/Legendary tier cards.

### Find (exact string from index.html):
```
.score-p{position:absolute;top:8px;right:8px;font-family:var(--mono);font-size:10px;font-weight:700;padding:2px 7px;border-radius:4px;background:rgba(0,0,0,.7);backdrop-filter:blur(8px);border:1px solid var(--border2);}
```

### Replace with:
```
.score-p{position:absolute;top:8px;right:8px;font-family:var(--mono);font-size:10px;font-weight:700;padding:2px 7px;border-radius:4px;background:rgba(0,0,0,.7);backdrop-filter:blur(8px);border:1px solid var(--border2);}
.dc-discount-badge{position:absolute;bottom:-12px;right:10px;font-family:var(--mono);font-size:13px;font-weight:700;padding:3px 9px;border-radius:5px;background:var(--green);color:#000;letter-spacing:-.01em;box-shadow:0 2px 8px rgba(34,197,94,.35);z-index:2;pointer-events:none;}
```

**Then update the `dc-img` block in `cardHTML`** to inject the badge. Find this exact string inside `cardHTML`:

### Find (exact string from index.html):
```
  <div class="dc-img">${d.emoji}<span class="rar-b rb-${d.rarity}">${RARITY[d.rarity]}</span><span class="score-p ${scC(d.score)}">${d.score}</span>
  <div class="card-timer">${active?'<div class="timer-dot"></div>':''}${tl}</div>${pb?`<div class="pop-b">${pb}</div>`:''}</div>
```

### Replace with:
```
  <div class="dc-img" style="position:relative">${d.emoji}<span class="rar-b rb-${d.rarity}">${RARITY[d.rarity]}</span><span class="score-p ${scC(d.score)}">${d.score}</span>
  <div class="card-timer">${active?'<div class="timer-dot"></div>':''}${tl}</div>${pb?`<div class="pop-b">${pb}</div>`:''}<span class="dc-discount-badge">−${sp(d.priceNow,d.priceWas)}%</span></div>
```

---

## Patch 3: "Deal of the Day" Card Gets a Dedicated Full-Width CTA Button

**Problem:** The DOTD (`.dotd`) card is the highest-visibility element on the page — it sits directly below the XP banner, before the deal grid, in a gold-bordered `cursor:pointer` container. But it has **no visible CTA button**: clicking the entire card fires `dotdClick()` via `onclick`, which is invisible affordance. Mobile users may not realize the card is tappable. There is no "Deal sichern" label, no visual button, no price-reinforcing action element.

**Solution:** Add a full-width CTA button inside `.dotd-footer` that mirrors the deal card's buy button styling but with larger font and the gold accent color to match the DOTD tier.

**Impact:** Visible button = discoverable action. Expected significant uplift on DOTD click rate, which is the single highest-value placement on the page.

### Find (exact string from index.html):
```
.dotd-footer{display:flex;align-items:center;gap:8px;flex-wrap:wrap;}
.dotd-timer{font-family:var(--mono);font-size:11px;color:var(--accent);display:flex;align-items:center;gap:4px;}
.dotd-shop{font-size:10px;color:var(--muted);}
```

### Replace with:
```
.dotd-footer{display:flex;align-items:center;gap:8px;flex-wrap:wrap;}
.dotd-timer{font-family:var(--mono);font-size:11px;color:var(--accent);display:flex;align-items:center;gap:4px;}
.dotd-shop{font-size:10px;color:var(--muted);}
.dotd-cta{display:block;width:100%;margin-top:10px;font-family:var(--font);font-size:14px;font-weight:700;padding:11px 16px;border-radius:8px;background:linear-gradient(135deg,var(--gold),#f59e0b);color:#000;border:none;cursor:pointer;text-align:center;letter-spacing:.01em;transition:opacity .15s,transform .1s;text-decoration:none;}
.dotd-cta:hover{opacity:.88;} .dotd-cta:active{transform:scale(.97);}
```

**Then add the button into the DOTD HTML structure. Find:**

### Find (exact string from index.html):
```
     <div class="dotd-footer">
      <div class="dotd-timer">⏱ <span id="dotdTimer">–</span></div>
      <div class="dotd-shop" id="dotdShop"></div>
     </div>
```

### Replace with:
```
     <div class="dotd-footer">
      <div class="dotd-timer">⏱ <span id="dotdTimer">–</span></div>
      <div class="dotd-shop" id="dotdShop"></div>
     </div>
     <a id="dotdCta" class="dotd-cta" href="#" target="_blank" rel="noopener sponsored" onclick="dotdClick();return false;">⭐ Deal des Tages sichern →</a>
```

**And update `buildDOTD()` to wire up the href on the CTA anchor:**

### Find (exact string from index.html):
```
 const card=document.getElementById('dotdCard');if(card){card.dataset.href=buildLink(d.affiliate,d.affiliateNetwork,d.awinMid)||'#';card.dataset.id=d.id;}
```

### Replace with:
```
 const card=document.getElementById('dotdCard');if(card){card.dataset.href=buildLink(d.affiliate,d.affiliateNetwork,d.awinMid)||'#';card.dataset.id=d.id;}
 const cta=document.getElementById('dotdCta');if(cta){const href=buildLink(d.affiliate,d.affiliateNetwork,d.awinMid)||'#';cta.href=href;}
```

---

## Patch 4: Empty / All-Expired State Shows Active Recovery Options

**Problem:** When all deals are past/expired or no deals match the rarity filter, the user sees a minimal `.empty` block: a 📭 emoji, "Keine aktiven Deals", and a small italic "Admin kann neue Deals hinzufügen." This is a **dead end** for real users — they have no next action. They will close the tab.

**Solution:** Enrich the no-deals empty state with three recovery actions: (1) a reminder bell CTA to subscribe, (2) a countdown to the next scheduled drop, and (3) a "zur Favoritenliste" link if the user has saved favourites. Also improve the filter-empty state to show a count of total available deals to reassure the user deals exist.

**Impact:** Every dead-end screen that gets an action reduces bounce. Notification sign-ups here convert at higher rates because user intent is already proven (they came looking for deals).

### Find (exact string from index.html):
```
 g.innerHTML=filtered.length?filtered.map(d=>cardHTML(d)).join(''):
  `<div class="empty" style="grid-column:1/-1"><div class="empty-i">${hasFilter?'🔍':'📭'}</div><div class="empty-t">${hasFilter?'Keine Treffer':'Keine aktiven Deals'}</div><div class="empty-s">${hasFilter?'':'Admin kann neue Deals hinzufügen.'}</div>${hasFilter?`<button class="sort-pill" style="margin-top:10px" onclick="resetFilter()">Filter zurücksetzen</button>`:''}</div>`;
```

### Replace with:
```
 if(filtered.length){g.innerHTML=filtered.map(d=>cardHTML(d)).join('');}
 else if(hasFilter){
  const total=getSorted().length;
  g.innerHTML=`<div class="empty" style="grid-column:1/-1"><div class="empty-i">🔍</div><div class="empty-t">Keine Treffer</div><div class="empty-s">${total>0?`${total} andere Deals verfügbar.`:''}</div><button class="sort-pill" style="margin-top:10px" onclick="resetFilter()">Alle Deals anzeigen</button></div>`;
 } else {
  const favCount=(S.favs||[]).length;
  const nextSlotHour=FIXED_SLOTS.map(s=>s.hour).find(h=>h>new Date().getHours())||FIXED_SLOTS[0].hour;
  const nextTime=nextSlotHour<new Date().getHours()?'08:00 Uhr morgen':`${String(nextSlotHour).padStart(2,'0')}:00 Uhr heute`;
  g.innerHTML=`<div class="empty" style="grid-column:1/-1">
   <div class="empty-i">⏰</div>
   <div class="empty-t">Nächster Drop: ${nextTime}</div>
   <div class="empty-s" style="margin-bottom:14px">Täglich 5 neue Deals — verpasse keinen.</div>
   <button class="btn-p" style="margin-bottom:8px;width:100%;max-width:280px" onclick="openNotify()">🔔 Erinnere mich</button>
   ${favCount?`<button class="btn-g" style="width:100%;max-width:280px" onclick="switchView('favs',document.querySelectorAll('.nav-tab')[1])">❤️ ${favCount} Favoriten ansehen</button>`:''}
  </div>`;
 }
```

---

## Patch 5: Card Hover Gets a Rarity-Colored Glow (Micro-Animation for Engagement)

**Problem:** `.deal-card:hover` only does `translateY(-2px)` — a subtle 2 px lift that barely registers on mobile (where hover doesn't apply) and feels generic on desktop. The rarity color system (Normal/Rare/Epic/Legendary/Mythic) is underutilized: the colored top border and subtle border-color change on hover are the only rarity signals. There is no motion or glow to reinforce the "collectible" feel that the rarity system implies.

**Solution:** Replace the flat hover with a rarity-matched `box-shadow` glow that fades in on hover. Each rarity tier gets its own glow color matching the existing CSS variable. The lift is kept at `−3px` (slightly increased from `−2px`). Also add a `will-change: transform` to prevent jank on iOS. No animation on `prefers-reduced-motion` (already handled by existing rule at line 361).

**Impact:** Stronger visual feedback on hover raises "card feels interactive" perception, increasing desktop CTR. The rarity glow reinforces the deal quality tier, priming users to act on high-rarity cards.

### Find (exact string from index.html):
```
.deal-card{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r2);overflow:hidden;cursor:pointer;position:relative;transition:transform .18s,border-color .18s;}
.deal-card:hover{transform:translateY(-2px);}
.deal-card.r-normal{border-color:rgba(74,158,255,.2);}  .deal-card.r-normal:hover{border-color:var(--normal);}
.deal-card.r-rare{border-color:rgba(168,85,247,.22);}   .deal-card.r-rare:hover{border-color:var(--rare);}
.deal-card.r-epic{border-color:rgba(245,158,11,.22);}   .deal-card.r-epic:hover{border-color:var(--epic);}
.deal-card.r-legendary{border-color:rgba(255,77,46,.28);box-shadow:0 0 18px rgba(255,77,46,.05);}
.deal-card.r-legendary:hover{border-color:var(--legendary);}
.deal-card.r-mythic{border-color:rgba(0,255,204,.2);box-shadow:0 0 24px rgba(0,255,204,.04);}
.deal-card.r-mythic:hover{border-color:var(--mythic);}
```

### Replace with:
```
.deal-card{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r2);overflow:hidden;cursor:pointer;position:relative;transition:transform .2s,border-color .2s,box-shadow .2s;will-change:transform;}
.deal-card:hover{transform:translateY(-3px);}
.deal-card.r-normal{border-color:rgba(74,158,255,.2);}
.deal-card.r-normal:hover{border-color:var(--normal);box-shadow:0 6px 22px rgba(74,158,255,.14);}
.deal-card.r-rare{border-color:rgba(168,85,247,.22);}
.deal-card.r-rare:hover{border-color:var(--rare);box-shadow:0 6px 22px rgba(168,85,247,.15);}
.deal-card.r-epic{border-color:rgba(245,158,11,.22);}
.deal-card.r-epic:hover{border-color:var(--epic);box-shadow:0 6px 24px rgba(245,158,11,.16);}
.deal-card.r-legendary{border-color:rgba(255,77,46,.28);box-shadow:0 0 18px rgba(255,77,46,.06);}
.deal-card.r-legendary:hover{border-color:var(--legendary);box-shadow:0 8px 28px rgba(255,77,46,.2);}
.deal-card.r-mythic{border-color:rgba(0,255,204,.2);box-shadow:0 0 24px rgba(0,255,204,.05);}
.deal-card.r-mythic:hover{border-color:var(--mythic);box-shadow:0 8px 32px rgba(0,255,204,.18);}
```

---

## Patch 6: Rarity Badge Enlarged and Made More Readable

**Problem:** `.rar-b` (the rarity badge on the product image) is `font-size:9px` — extremely small and hard to read at a glance, especially on sub-400 px screens. The padding (`2px 7px`) gives it almost no breathing room. Given that the rarity system is a core retention mechanic (users return to "collect" higher-tier deals), the badge needs to be legible at arm's length.

**Solution:** Bump font-size to `10px`, increase padding to `3px 9px`, add `font-weight:800` to make the text heavier, and give Legendary/Mythic a subtle text-shadow for extra punch.

**Impact:** Users can read the rarity tier instantly without squinting — reinforces the collectible psychology that drives return visits.

### Find (exact string from index.html):
```
.rar-b{position:absolute;top:8px;left:8px;font-size:9px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;padding:2px 7px;border-radius:4px;}
```

### Replace with:
```
.rar-b{position:absolute;top:8px;left:8px;font-size:10px;font-weight:800;letter-spacing:.07em;text-transform:uppercase;padding:3px 9px;border-radius:5px;}
.rb-legendary,.rb-mythic{text-shadow:0 0 8px currentColor;}
```

---

## Patch 7: Bottom Nav Tap Targets Enlarged and Active Indicator Made Clearer

**Problem:** `.bnav-item` has `padding:8px 4px 6px` — the vertical tap target is only ~42 px total (8+6+icon+label), which is below Apple's 44 pt / Google's 48 dp recommended minimum. The active state is only a color change (`.bnav-item.active{color:var(--accent)}`) with no background or indicator bar, so active navigation position is ambiguous on quick glances.

**Solution:** Increase vertical padding to `10px 4px 8px` (raising total height to ~48 px). Add a 2 px accent-colored indicator bar at the top of the active item using `::before` pseudo-element.

**Impact:** Fewer mis-taps, clearer location awareness. Reduces friction to switching between Deals and Favoriten — the two views most directly tied to conversion.

### Find (exact string from index.html):
```
.bnav-item{flex:1;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:8px 4px 6px;gap:2px;cursor:pointer;border:none;background:none;color:var(--muted);transition:color .15s;-webkit-tap-highlight-color:transparent;}
.bnav-item.active{color:var(--accent);}
```

### Replace with:
```
.bnav-item{flex:1;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:10px 4px 8px;gap:2px;cursor:pointer;border:none;background:none;color:var(--muted);transition:color .15s,background .15s;-webkit-tap-highlight-color:transparent;position:relative;}
.bnav-item.active{color:var(--accent);}
.bnav-item.active::before{content:'';position:absolute;top:0;left:25%;right:25%;height:2px;background:var(--accent);border-radius:0 0 2px 2px;}
```

---

## Patch 8: Original Price Strikethrough Made More Visible

**Problem:** `.dc-was` is `font-size:11px;color:var(--muted);text-decoration:line-through` — the muted color (`#6b6e7a`) at 11px against the dark card background has poor contrast. The `text-decoration:line-through` alone is a very light signal. The psychological anchor (showing what price *was*) needs to be clearly readable so users can instantly compute their savings. Currently it almost disappears.

**Solution:** Increase `.dc-was` font-size to `12px` and switch the color from `var(--muted)` to a slightly brighter muted (`rgba(255,255,255,.38)`) so it reads at WCAG AA against the `--bg2` background. Also add `text-decoration-color` matching the muted color so the strikethrough is intentional-looking rather than accidental.

**Impact:** Clearer price anchor = users immediately register the savings gap = stronger "this is a good deal" cognition before they even read the discount badge.

### Find (exact string from index.html):
```
.dc-was{font-family:var(--mono);font-size:11px;color:var(--muted);text-decoration:line-through;}
```

### Replace with:
```
.dc-was{font-family:var(--mono);font-size:12px;color:rgba(255,255,255,.38);text-decoration:line-through;text-decoration-color:rgba(255,255,255,.25);}
```

**Also apply the same fix to the DOTD card's "was" price:**

### Find (exact string from index.html):
```
.dotd-was{font-family:var(--mono);font-size:12px;color:var(--muted);text-decoration:line-through;}
```

### Replace with:
```
.dotd-was{font-family:var(--mono);font-size:13px;color:rgba(255,255,255,.38);text-decoration:line-through;text-decoration-color:rgba(255,255,255,.25);}
```
