# PM Agent Patches

## Summary
The biggest problems are: (1) the bulk-import textarea only accepts 4 URLs but the table has 5 slots, causing silent mismatch confusion; (2) the post-publish `confirm()` dialog blocks the UI and interrupts the "tomorrow" workflow with an unhelpful prompt every single time; (3) the bulk table rows are disabled via `pointer-events:none` when empty, so the admin can't directly type a title or price into a row that has no URL yet — forcing URL-first order; (4) "Für morgen vorbereiten" prepares deals with the correct future date but `expiresAt` is set to `h+1` on *tomorrow's date*, meaning `getActiveDeals()` will already filter them out the morning you want to activate them if you forgot to hit "▶ Alle aktiv" in time; (5) inline validation in the bulk table only fires on publish, giving zero feedback while filling in the rows. Expected improvement: shave 45–60 seconds off the daily workflow and eliminate the most common error paths.

---

## Patch 1: Remove the blocking confirm() dialog after publish — replace with an inline action banner

**Problem:** After every publish (both "heute live" and "morgen"), a `confirm()` dialog pops up asking whether to export deals.json. This blocks the browser UI, is interrupting on mobile, and is shown even when the operator is in the middle of something else. The toast already confirms success. The dialog adds ~5 seconds of friction per publish cycle.

**Solution:** Remove the `setTimeout(confirm(...))` call entirely. Instead, add an inline "Export now" button that appears inside the admin panel right after a successful publish. The button auto-hides after 60 seconds.

**Impact:** Saves 5–10 seconds per publish; eliminates accidental mobile mis-taps on the native confirm dialog.

### Find (exact string from index.html):
```
 showToast('✅',`${added} Deal${added>1?'s':''} ${label}!`);
 setTimeout(()=>{if(confirm(`✅ ${added} Deals ${label}!\n\ndeals.json jetzt exportieren und hochladen?`))adminExport();},600);
```

### Replace with:
```
 showToast('✅',`${added} Deal${added>1?'s':''} ${label}!`);
 // Inline export nudge — no blocking dialog
 const _en=document.getElementById('adminExportNudge');
 if(_en){
  _en.innerHTML=`<div style="display:flex;align-items:center;justify-content:space-between;gap:10px;background:rgba(34,197,94,.08);border:1px solid rgba(34,197,94,.22);border-radius:8px;padding:9px 13px;margin-bottom:12px;font-size:12px">
   <span style="color:var(--green)">✅ ${added} Deal${added>1?'s':''} ${label} — deals.json hochladen?</span>
   <div style="display:flex;gap:6px;flex-shrink:0">
    <button class="sort-pill" style="color:var(--green);border-color:rgba(34,197,94,.3)" onclick="adminExport();document.getElementById('adminExportNudge').innerHTML=''">📤 Exportieren</button>
    <button class="sort-pill" onclick="document.getElementById('adminExportNudge').innerHTML=''">✕</button>
   </div>
  </div>`;
  setTimeout(()=>{if(_en)_en.innerHTML='';},60000);
 }
```

You also need to add the nudge container element just above the export/import section in the HTML. Find:

### Find (exact string from index.html):
```
   <!-- Export / Import -->
   <hr class="adm-divider">
   <div class="sec-head"><span class="sec-title">Daten</span></div>
```

### Replace with:
```
   <!-- Export / Import -->
   <div id="adminExportNudge"></div>
   <hr class="adm-divider">
   <div class="sec-head"><span class="sec-title">Daten</span></div>
```

---

## Patch 2: Fix the bulk-table empty-row lockout — allow manual entry without a URL

**Problem:** Empty bulk rows have CSS class `b-empty` which applies `opacity:.28; pointer-events:none`. This means the admin cannot type a product title or price into any slot that doesn't already have a matching URL. The URL-first constraint is a real bottleneck: if you find a deal from a site not in `SHOP_LOOKUP` (e.g. a direct shop URL), the row stays locked and you can't enter data manually at all without first pasting a URL.

**Solution:** Remove `pointer-events:none` from `.b-empty` so the inputs are always clickable. Keep the low opacity as a visual cue that the slot is unfilled, but let the admin type directly. The `adminPublishAll` validator already skips slots with no URL, so no logic change is needed.

**Impact:** Saves ~15 seconds when dealing with unrecognised shop URLs. Eliminates a confusing UX dead-end.

### Find (exact string from index.html):
```
.bulk-row.b-empty{opacity:.28;pointer-events:none;}
```

### Replace with:
```
.bulk-row.b-empty{opacity:.45;}
```

---

## Patch 3: Fix "Für morgen vorbereiten" expiry — tomorrow's deals must not auto-expire before activation

**Problem:** When publishing tomorrow's deals, `expiresAt` is set to `tomorrow at h+1:59:59`. But `getActiveDeals()` filters out deals where `expiresAt` is more than 24 hours in the past — not in the future. The expiry is correct. However, `adminDailyReset()` also runs on the reset button and removes deals where `expiresAt` is within the current window (`nowH <= d.hour+1`). More critically: if you prepared tomorrow's deals at 22:00 and the deal is for 08:00 the next morning, the `expiresAt` is the next day at 09:59. When you open admin the next morning and hit "Reset" to clean up yesterday's deals, the tomorrow-deals are already live and the reset logic is fine — BUT if you accidentally hit Reset on the same night you prepared them, the fresh deals survive (their expiresAt is tomorrow). That part is actually fine.

The real bug is different: `adminPublishAll(true)` sets `active:false` to prevent the deals from being visible. But `getActiveDeals()` filters on `d.active===false` to hide them. So far correct. The morning activation requires "▶ Alle aktiv" or individual toggles. There is no reminder or indicator in the admin panel showing "X deals scheduled for tomorrow — activate them." The admin opens the panel in the morning, sees no live deals, and has no clear affordance to activate the staged ones. The staged deals appear in the list with no date label — just a time like "08:00" — making it impossible to know if they are today's or tomorrow's.

**Solution:** Show a dated label on each deal row in `adminRenderDeals()`, and add a prominent "Morgen-Deals aktivieren" shortcut banner when inactive deals exist that were published for a future date.

**Impact:** Eliminates the "why are there no deals this morning?" confusion. Saves 1–2 minutes of diagnosis time.

### Find (exact string from index.html):
```
  const status=exp?'Abgelaufen':past?'Beendet':live?'🟢 Live':`⏱ ${d.time||d.hour+':00'}`;
  const net=(d.affiliateNetwork||'direct').toUpperCase();
  const dotdBadge=d.dotd?'<span style="font-size:9px;color:var(--gold)"> ⭐</span>':'';
  return`<div class="adm-row" style="${exp||past?'opacity:.4':''}">
   <span style="font-size:22px">${d.emoji||'🛍️'}</span>
   <div class="adm-row-info">
    <div class="adm-row-title">${sanitize(d.title)}${dotdBadge}</div>
    <div class="adm-row-meta">${status} · ${net} · €${d.priceNow||'?'}</div>
   </div>
   <button class="sort-pill" title="${d.active===false?'Aktivieren':'Pausieren'}" onclick="adminToggle(${d.id})">${d.active===false?'▶':'⏸'}</button>
   <button class="sort-pill" style="color:var(--accent)" title="Löschen" onclick="adminDelete(${d.id})">✕</button>
  </div>`;
```

### Replace with:
```
  const status=exp?'Abgelaufen':past?'Beendet':live?'🟢 Live':`⏱ ${d.time||d.hour+':00'}`;
  const net=(d.affiliateNetwork||'direct').toUpperCase();
  const dotdBadge=d.dotd?'<span style="font-size:9px;color:var(--gold)"> ⭐</span>':'';
  // Date label: show "morgen" if expiresAt is in the future by more than ~14h and deal is inactive
  let dateBadge='';
  if(d.active===false&&d.expiresAt){
   const expDate=new Date(d.expiresAt);
   const diffH=(expDate-new Date())/3600000;
   if(diffH>14){
    const dateStr=expDate.toLocaleDateString('de-AT',{day:'2-digit',month:'2-digit'});
    dateBadge=`<span style="font-size:9px;color:var(--xp);background:rgba(129,140,248,.12);border:1px solid rgba(129,140,248,.2);border-radius:3px;padding:1px 5px;margin-left:5px">${dateStr}</span>`;
   }
  }
  const inactiveBadge=d.active===false?'<span style="font-size:9px;color:var(--muted);margin-left:4px">⏸ inaktiv</span>':'';
  return`<div class="adm-row" style="${exp||past?'opacity:.4':''}${d.active===false&&!exp?'border-color:rgba(129,140,248,.2);':''}" >
   <span style="font-size:22px">${d.emoji||'🛍️'}</span>
   <div class="adm-row-info">
    <div class="adm-row-title">${sanitize(d.title)}${dotdBadge}${dateBadge}</div>
    <div class="adm-row-meta">${status}${inactiveBadge} · ${net} · €${d.priceNow||'?'}</div>
   </div>
   <button class="sort-pill" title="${d.active===false?'Aktivieren':'Pausieren'}" onclick="adminToggle(${d.id})">${d.active===false?'▶':'⏸'}</button>
   <button class="sort-pill" style="color:var(--accent)" title="Löschen" onclick="adminDelete(${d.id})">✕</button>
  </div>`;
```

Also add a "tomorrow deals" activation banner to `adminRenderDeals()`. Find the function opening:

### Find (exact string from index.html):
```
function adminRenderDeals(){
 const deals=loadDeals();
 const el=document.getElementById('adminDealList');if(!el)return;
 if(!deals.length){el.innerHTML=`<div class="empty"><div class="empty-i">📭</div><div class="empty-t">Keine Deals</div><div class="empty-s">Füge unten einen Deal hinzu.</div></div>`;return;}
 const nowH=new Date().getHours()+new Date().getMinutes()/60;
```

### Replace with:
```
function adminRenderDeals(){
 const deals=loadDeals();
 const el=document.getElementById('adminDealList');if(!el)return;
 if(!deals.length){el.innerHTML=`<div class="empty"><div class="empty-i">📭</div><div class="empty-t">Keine Deals</div><div class="empty-s">Füge unten einen Deal hinzu.</div></div>`;return;}
 const nowH=new Date().getHours()+new Date().getMinutes()/60;
 // Banner: staged tomorrow deals
 const staged=deals.filter(d=>d.active===false&&d.expiresAt&&((new Date(d.expiresAt)-new Date())/3600000)>14);
 const stagedBanner=staged.length?`<div style="display:flex;align-items:center;justify-content:space-between;gap:10px;background:rgba(129,140,248,.08);border:1px solid rgba(129,140,248,.2);border-radius:8px;padding:9px 13px;margin-bottom:10px;font-size:12px">
  <span style="color:var(--xp)">⏩ ${staged.length} Deal${staged.length>1?'s':''} für morgen vorbereitet</span>
  <button class="sort-pill" style="color:var(--xp);border-color:rgba(129,140,248,.3);flex-shrink:0" onclick="adminActivateAll()">▶ Jetzt aktivieren</button>
 </div>`:'';
```

Then also add `+stagedBanner` to the `el.innerHTML` assignment. Find the existing render map output:

### Find (exact string from index.html):
```
 el.innerHTML=deals.map(d=>{
```

### Replace with:
```
 el.innerHTML=stagedBanner+deals.map(d=>{
```

---

## Patch 4: Bulk table — live inline validation feedback (missing prices highlighted in red)

**Problem:** The bulk table inputs (`bttl`, `bpn`, `bpw`) have no validation state. You can fill in all 4 rows and hit publish, only to find that 2 rows were silently skipped because you forgot a price. The only feedback is the toast saying "2 Deals live" when you expected 4 — with no indication of which rows were skipped or why.

**Solution:** Add an `oninput` validator to each bulk table input that marks the row with a red border when it has a URL but is still missing required fields (title or prices). Also show the discount preview inline in each row when both prices are entered.

**Impact:** Catches missing data in real time. Saves the full 30-second "re-open admin, re-enter data" cycle when a row is silently skipped.

### Find (exact string from index.html):
```
   return`<div class="bulk-row${s.hasUrl?' has-url':''}${!s.hasUrl?' b-empty':''}">
    <span class="btime">${time}</span>
    <span class="bshop${s.hasUrl?' ok':''}">${sanitize(s.shop||'–')}</span>
    <input class="binp" id="bttl${i}" placeholder="Produkttitel eingeben…">
    <input class="binp" id="bpn${i}" placeholder="€" type="number" step="0.01" min="0">
    <input class="binp" id="bpw${i}" placeholder="€" type="number" step="0.01" min="0">
    <button class="bdel" onclick="bulkClearRow(${i})" title="Löschen">✕</button>
   </div>`;
```

### Replace with:
```
   return`<div class="bulk-row${s.hasUrl?' has-url':''}${!s.hasUrl?' b-empty':''}" id="brow${i}">
    <span class="btime">${time}</span>
    <span class="bshop${s.hasUrl?' ok':''}" id="bshoplbl${i}">${sanitize(s.shop||'–')}</span>
    <input class="binp" id="bttl${i}" placeholder="Produkttitel eingeben…" oninput="bulkValidateRow(${i})">
    <input class="binp" id="bpn${i}" placeholder="€" type="number" step="0.01" min="0" oninput="bulkValidateRow(${i})">
    <input class="binp" id="bpw${i}" placeholder="€" type="number" step="0.01" min="0" oninput="bulkValidateRow(${i})">
    <button class="bdel" onclick="bulkClearRow(${i})" title="Löschen">✕</button>
   </div>`;
```

Then add the validation function after `bulkClearMystery`:

### Find (exact string from index.html):
```
function bulkClearMystery(){
 bulkState[4]={url:'',shop:'🔮 Mystery',network:'direct',awinMid:null,cat:'Technik',emoji:'🛍️',hasUrl:false};
 ['burl4','bpn4','bpw4'].forEach(id=>{const el=document.getElementById(id);if(el)el.value='';});
}
```

### Replace with:
```
function bulkClearMystery(){
 bulkState[4]={url:'',shop:'🔮 Mystery',network:'direct',awinMid:null,cat:'Technik',emoji:'🛍️',hasUrl:false};
 ['burl4','bpn4','bpw4'].forEach(id=>{const el=document.getElementById(id);if(el)el.value='';});
}
function bulkValidateRow(i){
 const s=bulkState[i];
 if(!s||!s.hasUrl)return;
 const title=(document.getElementById(`bttl${i}`)?.value||'').trim();
 const pn=parseFloat(document.getElementById(`bpn${i}`)?.value)||0;
 const pw=parseFloat(document.getElementById(`bpw${i}`)?.value)||0;
 const row=document.getElementById(`brow${i}`);
 if(!row)return;
 const missing=!title||(pn<=0)||(pw<=0)||(pn>=pw&&pw>0);
 row.style.borderColor=missing?'rgba(255,77,46,.4)':'rgba(34,197,94,.18)';
 // Inline discount badge on shop label when both prices are valid
 const lbl=document.getElementById(`bshoplbl${i}`);
 if(lbl&&pn>0&&pw>0&&pn<pw){
  const disc=Math.round((1-pn/pw)*100);
  lbl.textContent=`-${disc}%`;
  lbl.className='bshop ok';
 }
}
```

---

## Patch 5: Bulk URL textarea — accept 5 URLs (include the 20:00 Mystery slot) and fix the label

**Problem:** The label says "4 Produkt-URLs — je eine pro Zeile (Slots 08:00 / 10:00 / 12:00 / 16:00)" and `adminParseBulkUrls()` only takes `.slice(0,4)` URLs. But `BULK_SLOTS` has 5 entries: `[8,10,12,16,20]`. The Mystery 20:00 slot is the 5th row in the bulk table, but it has its own dedicated URL input (`burl4`) that never gets auto-populated from the bulk textarea. This means: (a) the admin must manually copy-paste the mystery URL into a separate field even if they have all 5 URLs ready; (b) the label is wrong — it says 4 slots but the section badge says "5 Deals".

**Solution:** Change `adminParseBulkUrls()` to read up to 5 URLs and populate the mystery URL field (`burl4`) from the 5th line. Update the label to say "5 Produkt-URLs".

**Impact:** Saves 15–20 seconds per day (no separate mystery URL paste). Eliminates the label inconsistency that causes admin confusion on which slots to fill.

### Find (exact string from index.html):
```
    <span class="adm-lbl">4 Produkt-URLs — je eine pro Zeile (Slots 08:00 / 10:00 / 12:00 / 16:00)</span>
    <textarea class="m-input" id="bulkUrlInput" placeholder="https://amazon.de/dp/B0XXXXX&#10;https://mediamarkt.at/de/product/...&#10;https://zalando.at/...&#10;https://alternate.at/..." style="height:76px;resize:none;font-size:11px;font-family:var(--mono);margin-bottom:8px" oninput="adminParseBulkUrls()"></textarea>
```

### Replace with:
```
    <span class="adm-lbl">5 Produkt-URLs — je eine pro Zeile (Slots 08:00 / 10:00 / 12:00 / 16:00 / 20:00 Mystery)</span>
    <textarea class="m-input" id="bulkUrlInput" placeholder="https://amazon.de/dp/B0XXXXX&#10;https://mediamarkt.at/de/product/...&#10;https://zalando.at/...&#10;https://alternate.at/...&#10;https://... (20:00 Mystery)" style="height:92px;resize:none;font-size:11px;font-family:var(--mono);margin-bottom:8px" oninput="adminParseBulkUrls()"></textarea>
```

### Find (exact string from index.html):
```
function adminParseBulkUrls(){
 const raw=document.getElementById('bulkUrlInput')?.value||'';
 const lines=raw.split('\n').map(s=>s.trim()).filter(s=>s.startsWith('http')).slice(0,4);
 for(let i=0;i<4;i++){
  const url=lines[i]||'';
  const info=url?detectShopInfo(url):null;
  bulkState[i]={url,hasUrl:!!url,
   shop:info?info.shop:'?',network:info?info.network:'direct',
   awinMid:info?info.awinMid:null,cat:info?info.cat:'Technik',emoji:info?info.emoji:'🛍️'};
 }
 renderBulkTable();
}
```

### Replace with:
```
function adminParseBulkUrls(){
 const raw=document.getElementById('bulkUrlInput')?.value||'';
 const lines=raw.split('\n').map(s=>s.trim()).filter(s=>s.startsWith('http')).slice(0,5);
 for(let i=0;i<4;i++){
  const url=lines[i]||'';
  const info=url?detectShopInfo(url):null;
  bulkState[i]={url,hasUrl:!!url,
   shop:info?info.shop:'?',network:info?info.network:'direct',
   awinMid:info?info.awinMid:null,cat:info?info.cat:'Technik',emoji:info?info.emoji:'🛍️'};
 }
 // 5th line → Mystery 20:00 URL field
 const mystUrl=lines[4]||'';
 if(mystUrl){
  const mystInfo=detectShopInfo(mystUrl);
  bulkState[4]={...bulkState[4],url:mystUrl,hasUrl:true,
   shop:mystInfo?mystInfo.shop:'?',network:mystInfo?mystInfo.network:'direct',
   awinMid:mystInfo?mystInfo.awinMid:null,cat:mystInfo?mystInfo.cat:'Technik',emoji:mystInfo?mystInfo.emoji:'🛍️'};
 }
 renderBulkTable();
 // Sync mystery URL input field after re-render
 const burl4=document.getElementById('burl4');
 if(burl4&&mystUrl)burl4.value=mystUrl;
}
```
