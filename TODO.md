# DailyFive AT — TODO

---

## Heute erledigt (2026-06-12)

- [x] GitHub Repository + Auto-Deploy via GitHub Actions → Netlify
- [x] Live-URL verifiziert (dailyfive-at.netlify.app, HTTP 200)
- [x] 10 QA-Bugs gefunden und behoben (Countdowns, DOTD, Affiliate-Links, Impressum)
- [x] deals.json mit echten Amazon.de Produkt-Links (5 Deals, Produkt-Level URLs)
- [x] Netlify Forms: E-Mail-Anmeldungen landen im Netlify Dashboard
- [x] Browser Push Notifications: Permission-Request + lokales Deal-Scheduling
- [x] Cloudflare Analytics: Dediziertes Token-Feld im Admin
- [x] Bell-Icon: zeigt Push-Status (grün = Push aktiv, rot = E-Mail)
- [x] Impressum: Raheel Ahmed, Wien, Österreich (keine Adresse)
- [x] Mobile Layout verbessert (stats-grid, Bottom Nav)

---

## Was jetzt live ist

- Vollständige PWA unter https://dailyfive-at.netlify.app
- 5 echte Deals mit Amazon.de Links (Samsung Buds3 Pro, Philips Airfryer, Anker MagGo, De'Longhi, Mystery AirPods Pro)
- E-Mail-Liste baut sich auf (Netlify Dashboard → Forms)
- Push Notifications funktionieren (Browser-Permission → SW-Scheduling)
- Admin-Panel unter /index.html#admin (Passwort: dailyfive2026)

---

## Offen / Bekannte Einschränkungen

- [ ] **Cloudflare Token eintragen** — Admin öffnen → CF Token eingeben → Speichern
- [ ] **Deals täglich aktualisieren** — jeden Morgen neue Deals via Admin-Schnellimport
- [ ] **Amazon Affiliate-Konto** — partnernet.amazon.de anmelden für Provision-Tracking
- [ ] **PNG Icons** — manifest.json hat nur SVG; ältere iOS-Versionen brauchen PNG für PWA-Icon
- [ ] **Echter Push ohne Page-Open** — aktuell: lokale Notifications (Page muss offen sein); für echte Web-Push: VAPID-Backend nötig
- [ ] **Preise täglich prüfen** — Amazon-Preise ändern sich; deals.json manuell oder via Skript updaten

---

## Nächste 3 Schritte (höchster Impact)

### 1. Deals täglich aktualisieren (jeden Morgen, ~10 Min)
- Admin öffnen: dailyfive-at.netlify.app/#admin
- Passwort: dailyfive2026
- "⚡ Tagesplan" → 5 Amazon/Shop URLs einfügen → Preise/Titel ausfüllen → Speichern
- Danach: deals.json auf GitHub aktualisieren (oder direkt im Admin-Editor)
- **Warum**: Veraltete Deals zerstören Vertrauen. Tägliche Updates = tägliche Nutzer.

### 2. Cloudflare Web Analytics aktivieren
- dash.cloudflare.com → Web Analytics → Add a site → Token kopieren
- Admin → "📊 Cloudflare Web Analytics" → Token einfügen → Speichern
- **Warum**: Ohne Analytics gibt es keine Entscheidungsgrundlage. Kostenlos, DSGVO-konform.

### 3. Erste Nutzer gewinnen (10–50 echte Nutzer)
- WhatsApp/Telegram-Kanal erstellen: "DailyFive AT — 5 Deals täglich"
- Bekanntenkreis einladen
- Reddit: r/Austria, r/Finanzen_AT — Deal-Link posten (mit echter Empfehlung, kein Spam)
- **Warum**: Echte Nutzer zeigen wo es noch hakt. Kein Feature ist wertvoller als echtes Feedback.

---

## Backlog (wenn Nutzer da sind)

- Referral-System (Freunde einladen → XP)
- Tägliche E-Mail-Automation (Brevo/Mailchimp)
- Amazon Affiliate-Tracking verifizieren
- WhatsApp/Telegram-Kanal aufbauen
- Sponsored Deals (B2B-Revenue)
