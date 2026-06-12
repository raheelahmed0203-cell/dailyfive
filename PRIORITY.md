# DailyFive — Startup-Analyse & Prioritäten

## Ehrliche Bewertung

### Monetarisierung: 2/10 (aktuell)

**Das Kernproblem:** Alle Affiliate-Links zeigen auf Shop-Homepages (`mediamarkt.at/?ref=dailyfive`), nicht auf Produktseiten.
Ein Klick auf die Homepage konvertiert mit ~0.1%. Ein Klick auf das Produktlisting mit ~4-8%.
Das bedeutet: DailyFive verdient aktuell **praktisch nichts**, obwohl Nutzer auf "Deal sichern" klicken.

**Das Modell funktioniert** — aber erst wenn:
- Links auf echte Produktseiten zeigen
- Affiliate-Netzwerk korrekt eingebunden ist (AWIN, TradeTracker AT, Amazon Partnernet)
- E-Mail-Liste tatsächlich irgendwo landet (aktuell nur localStorage)

Hochrechnung bei 500 DAU: 5 Deals × 50 Klicks × 3% Conversion × €4 Avg-Commission = **€30/Tag = €900/Monat**.
Bei 5.000 DAU: **€9.000/Monat**. Das Modell skaliert linear mit Traffic.

---

### Nutzerbindung: 7/10

**Stärken:** Streak-System, tägliche Belohnungen, FOMO-Countdowns — das ist besser als 90% der österreichischen Deal-Seiten.
**Schwäche:** E-Mail-Benachrichtigungen funktionieren nicht (kein Backend), Push-Notifications haben keinen Server. Wenn ein Nutzer die App schließt, gibt es keinen Weg, ihn zurückzuholen.

---

### Wachstumspotenzial: 8/10

**Österreich ist der richtige Markt** — klein genug um schnell #1 zu werden, groß genug für €20k+/Monat.
MyDealz/Pepper.at (der Platzhirsch) ist volumengetrieben und Community-abhängig. DailyFive ist kuriert + gamifiziert — das ist ein verteidigbarer Unterschied.
**Expansionspfad:** AT → DE → CH (selbe Sprache, 10x größerer Markt).

---

### Konkurrenz in Österreich

| Competitor | Modell | Schwäche | DailyFive-Vorteil |
|---|---|---|---|
| **MyDealz/Pepper.at** | Community-deals, 500k+ Besucher/Monat | Zu viel Rauschen, keine Kuration | 5/Tag = Vertrauen |
| **Geizhals.at** | Preisvergleich | Kein Editorial, kein FOMO | Emotion + Countdown |
| **Preisvergleich.at** | Preisvergleich | Langweilig, kein Engagement | Gamification |
| **Gutscheinpony.at** | Gutschein-Codes | Kein Scarcity-Mechanismus | Zeitdruck-Modell |
| **Zalando/Amazon Newsletter** | Brand-Shops | Interessen-Konflikt (eigene Produkte) | Plattform-unabhängig |

**Reales Risiko:** MyDealz könnte ein "DailyPicks"-Feature launchen. Timeline: 12-18 Monate wenn DailyFive Traktion zeigt. **Das Zeitfenster ist jetzt.**

---

## Prioritäten

> Reihenfolge: Was bringt zuerst Geld, dann Nutzer, dann läuft es von selbst.

---

## BLOCK 1 — Umsatz (macht ohne das keinen Sinn weiterzumachen)

### P1 · Echte Affiliate-Links implementieren
**Impact: €0 → €500+/Monat bei 200 DAU**

Die Seed-Deals haben Homepage-Links. Das muss auf Produkt-Links geändert werden.

Schritte:
1. Amazon Partnernet AT anmelden: `partnernet.amazon.de` — kostenlos, sofortige Genehmigung
2. AWIN-Account erstellen: `awin.com` — MediaMarkt AT, Saturn AT, Interspar sind dort
3. Für jeden Deal: Deep-Link auf das exakte Produkt (nicht Homepage)
4. `?ref=dailyfive` ersetzen durch echte Tracking-Parameter (`?tag=dailyfive-21` für Amazon)
5. Im Deal-Objekt: `affiliateNetwork: 'amazon'|'awin'|'direct'` Feld hinzufügen für späteres Tracking

Warum #1: Ohne das ist jeder Klick wertlos. Das ist der einzige Änderung mit sofortigem Geld-Effekt.

---

### P2 · E-Mail-Dienst anbinden (Brevo kostenlos bis 300 Mails/Tag)
**Impact: E-Mail-Liste wird zu echtem Asset, ermöglicht täglichen Traffic**

Aktuell: E-Mails landen in `localStorage`. Die Liste existiert nicht wirklich.

Schritte:
1. Brevo-Account erstellen (kostenlos, keine Kreditkarte): `brevo.com`
2. API-Key holen
3. `submitNotify()` in index.html: `fetch('https://api.brevo.com/v3/contacts', ...)` statt localStorage
4. Automatische Willkommens-Mail: "Erster Drop morgen 08:00"
5. Tägliche Mail um 07:45 mit den 5 heutigen Deals (manuell oder Zapier)

Warum #2: Eine E-Mail-Liste von 1.000 AT-Nutzern ist €2.000-5.000/Monat wert (Sponsored Newsletters).
MediaMarkt AT zahlt €50-200 pro Sponsored-Email-Slot bei relevanter Liste.

---

### P3 · Sponsored Deal / "Featured" Platzierung einbauen
**Impact: Direktes B2B-Revenue, €50-500 pro featured Deal**

Ein neues Feld `featured: true` im Deal-Objekt. Featured Deals:
- Erscheinen immer als #1 in der Liste
- Haben ein kleines "Gesponsert"-Label (rechtlich notwendig)
- Haben eigene Optik (goldener Border statt Rarity-System)

Dann: 5 österreichische Online-Shops anschreiben (klein anfangen: lokale Shops, nicht MediaMarkt).
Pitch: "5.000 kaufbereite Österreicher täglich, 24h Platzierung, €99 Flat."

---

### P4 · Conversion-Tracking einbauen
**Impact: Beweist den ROI für Shops, skaliert Sponsored Deals**

Aktuell: Niemand weiß, wie viele Nutzer wirklich kaufen.

```javascript
// Nach dem Klick auf "Deal sichern":
fetch('/track', { method: 'POST', body: JSON.stringify({ dealId, shop, price }) })
// Oder: UTM-Parameter auf allen Affiliate-Links, dann Google Analytics
```

Minimallösung ohne Backend: Google Analytics 4 + UTM-Parameter auf Affiliate-Links.
`https://affiliate-link.at?utm_source=dailyfive&utm_medium=deal&utm_campaign=deal-{id}`
→ Shops sehen ihren Traffic aus DailyFive in ihrem GA-Dashboard.

---

## BLOCK 2 — Nutzerwachstum (erst sinnvoll wenn P1+P2 erledigt)

### P5 · Referral-System
**Impact: Viraler Loop, organisches Wachstum ohne Werbekosten**

Mechanik: "Lade 3 Freunde ein → bekomme Mystery Drop 1h früher + 200 XP"

Implementation:
- Referral-Code generieren: `btoa(email).slice(0,8)` → kurzer Code
- URL: `dailyfive.at?ref=ABC123`
- Beim Besuch mit `?ref=` Parameter: Referrer kriegt 50 XP wenn Neuer sich anmeldet
- In-App anzeigen: "Du hast 2/3 Freunden eingeladen"

Warum jetzt noch nicht #1: Viraler Loop braucht ein Produkt, das Leute weiterempfehlen wollen. Erst Wert liefern (echte Deals), dann viral.

---

### P6 · Tägliche WhatsApp/Telegram-Kanäle
**Impact: 0-Klick-Retention, kein Login nötig, höchste Öffnungsrate aller Kanäle**

WhatsApp Broadcast-Kanal oder Telegram-Kanal (kostenlos):
- Täglich 07:55: "Heute 5 Drops 🔥: [Deal1] €X, [Deal2] €Y, ... → dailyfive.at"
- Einziger CTA: Website besuchen
- Wachstum über "Teile den Kanal mit Freunden"

Österreich-spezifisch: WhatsApp-Kanäle haben in AT höhere Adoption als Telegram.
Aufwand: 10 Min/Tag. Skaliert auf 10k+ Follower ohne Budget.

---

### P7 · SEO: Deal-Unterseiten für Google
**Impact: Passiver Traffic ohne Werbekosten**

Aktuell: DailyFive ist eine Single-Page-App. Google indexiert praktisch nichts.

Einfachste Lösung ohne Backend-Umbau:
- Statische HTML-Seiten für Kategorien: `technik-deals-oesterreich.html`, `kueche-deals-oesterreich.html`
- Jede Seite: 800 Wörter Text + Deals-Embed
- Ziel-Keywords: "günstige Technik AT", "Angebote heute Österreich"

Diese Keywords haben 500-5.000 monatliche Suchanfragen in AT bei niedriger Konkurrenz.

---

### P8 · OG-Tags & Deal-spezifische Share-Previews
**Impact: Jeder Share-Klick wird zu einer Werbeanzeige**

Aktuell: `<meta og:image>` fehlt. Wenn jemand einen Deal teilt, sieht der Empfänger keinen Preview.

```html
<meta property="og:title" content="DailyFive — 5 Deals. Täglich.">
<meta property="og:description" content="Samsung Galaxy Buds3 Pro für €129 statt €229 — nur heute!">
<meta property="og:image" content="https://dailyfive.at/og-image.png">
```

Dynamisch via URL-Parameter: `dailyfive.at?deal=1` → OG-Tags passen sich an (braucht serverseitiges Rendering oder einen Edge-Worker).

---

## BLOCK 3 — Automatisierung (erst sinnvoll wenn Nutzer da sind)

### P9 · Admin-Panel in der App selbst
**Impact: Deals täglich aktualisieren ohne Code-Kenntnisse**

Aktuell: Deals via `localStorage('df_admin_deals')` updaten — das ist manuell und fehleranfällig.

Einfaches passwortgeschütztes Admin-Panel:
- URL: `dailyfive.at#admin` + Passwort-Abfrage
- Formular: Deal hinzufügen/bearbeiten/löschen
- "Live Preview" direkt neben dem Formular
- Export: JSON-Download für Backup

Kein Backend nötig: Formular schreibt in localStorage. Für Multi-Device: Firebase Realtime DB Free Tier (1GB kostenlos).

---

### P10 · Deal-Feed aus Affiliate-Netzwerken (Produktkatalog-Import)
**Impact: Halbautomatische Deal-Erstellung, spart 1-2h täglich**

AWIN und Amazon bieten Produktdaten-Feeds als CSV/XML:
- Täglich automatisch herunterladen (Cron-Job oder Zapier)
- Filtern nach: Rabatt > 30%, Preis < €500, österreichische Verfügbarkeit
- Top 10 Kandidaten per E-Mail an Admin → Admin wählt 5 aus
- Kein vollautomatisches Publishing — kuratierte Qualität ist der USP

---

### P11 · Analytics-Dashboard für Shops (B2B SaaS-Potential)
**Impact: Rechtfertigt höhere Preise für Sponsored Deals**

Einfaches Dashboard: "Ihr Deal wurde X-mal geklickt, Y-mal geteilt, geschätzter Umsatz: €Z"
Zugang: Passwortgeschützter Link per E-Mail nach Deal-Ende.
Daten kommen aus: UTM-Parameter in Google Analytics.

Langfristig: Shops zahlen €200+/Monat für Premium-Placement + Reporting.

---

### P12 · Tägliche E-Mail automatisieren (Zapier/n8n)
**Impact: Spart 15 Min/Tag, skaliert auf 10k+ Subscriber ohne Mehraufwand**

Zapier-Flow (kostenlos bis 100 tasks/Monat):
1. Jeden Morgen 07:45: Google Sheets (manuelle Deals eingetragen) → Brevo-Template
2. Template: "Heute 5 Drops:" + automatische Deal-Karten aus Sheet-Daten
3. Senden an gesamte Liste

n8n (self-hosted, kostenlos) für mehr Kontrolle und höhere Volumen.

---

## Zusammenfassung

```
WOCHE 1-2:  P1 (echte Affiliate-Links) + P2 (Brevo E-Mail)
            → Erste Cent-Einnahmen, E-Mail-Liste wächst

WOCHE 3-4:  P5 (Referral) + P6 (Telegram/WhatsApp) + P8 (OG-Tags)  
            → Erste organische Verbreitung

MONAT 2:    P3 (Sponsored Deals) + P9 (Admin-Panel)
            → Erste B2B-Einnahmen, tägliches Updaten wird einfacher

MONAT 3+:   P4, P7, P10, P11, P12
            → Automatisierung, SEO-Traffic, skalierendes B2B

ZIEL 6 MONATE: €2.000-5.000/Monat (realistisch bei konsequenter Umsetzung)
ZIEL 12 MONATE: €10.000+/Monat (AT + DACH-Expansion)
```

---

> Das Modell funktioniert. Das Fenster in Österreich ist offen.
> Der einzige Fehler wäre, Zeit mit Features zu verbringen bevor P1 erledigt ist.
