---
name: shop-check
description: This skill should be used when the user asks to "check my online shop", "audit my e-commerce site", "is my shop legal", "Impressum prüfen", "Button-Lösung", "zahlungspflichtig bestellen", "Widerrufsbelehrung prüfen", "AGB prüfen", "PAngV", "Preisangaben MwSt", "Versandkosten angeben", "Widerrufsformular", "shop-recht", "e-commerce compliance Germany", "online shop German law", "§ 312j BGB", or asks whether a German online shop meets e-commerce legal requirements (DDG, BGB, PAngV).
version: 1.0.0
---

# /shop-check — E-Commerce Rechtsprüfung (Deutschland)

Prüft Online-Shops auf Konformität mit deutschen E-Commerce-Pflichten: Impressum, Button-Lösung, Preisangaben, Widerrufsrecht und AGB. Stützt alle Befunde auf aktuelle Gesetzestexte via `rechtsinformationen-bund-de` MCP.

**Rechtsrahmen:** DDG § 5, BGB §§ 305–306, 312g, 312j, 355–357, PAngV. Gilt für B2C-Online-Shops mit Sitz oder Zielpublikum in Deutschland.

---

## Step 0 — Route & Setup

Parse invocation args or user intent:

| Pattern | Modus |
|---|---|
| `scan <url>` oder URL gegeben | → **URL-Scan** (Live-Shop prüfen) |
| `audit <path>` oder Dateipfad | → **Code-Audit** (Checkout-Implementierung) |
| `draft <typ>` | → **Textbausteine** (Widerrufsbelehrung, AGB-Hinweise, Button-Texte) |
| Keine Args oder Freitext | → **Vollprüfung** (URL-Scan + Befund-Report + Angebot für Textbausteine) |

Check MCP availability: attempt `intelligente_rechtssuche` with query `BGB Fernabsatz Widerrufsrecht § 355`. If it responds, use it for every law lookup. If unavailable, fall back to built-in knowledge — note which source was used in each finding.

Display:
> **shop-check** | Modus: [X] | Ziel: [Y] | Gesetzesquelle: [MCP / Trainingsdaten]

---

## Modus A — URL-Scan

Prüft den Live-Shop gegen alle sieben Pflichtbereiche.

### A1 — Abruf

`WebFetch` auf die Ziel-URL (Startseite). Dann zusätzlich abrufen (als Footer-Links oder direkte Pfade):
- `/impressum` — Impressum
- `/widerruf` oder `/widerrufsbelehrung` oder `/widerrufsrecht` — Widerrufsbelehrung
- `/agb` oder `/allgemeine-geschaeftsbedingungen` — AGB
- Eine Produktdetailseite (Preisangaben prüfen)
- Checkout / Warenkorb / Bestellseite, falls öffentlich erreichbar (Button-Lösung prüfen)

Notiere alle gefundenen URLs. Extrahiere relevante Textpassagen für jeden Prüfbereich.

### A2 — Prüfmatrix

Klassifizierung: 🔴 Verstoß / 🟡 Risiko / 🟢 Konform / ⚪ Nicht automatisch prüfbar

---

**1. Impressum (DDG § 5)**

| Pflichtangabe | Rechtsgrundlage | Klassifizierung |
|---|---|---|
| Impressum-Seite vorhanden und erreichbar | DDG § 5 Abs. 1 | 🔴 falls fehlt |
| Vollständiger Name / Firmenname | DDG § 5 Abs. 1 Nr. 1 | 🔴 falls fehlt |
| Ladungsfähige Anschrift (kein Postfach) | DDG § 5 Abs. 1 Nr. 1 | 🔴 falls nur Postfach |
| E-Mail-Adresse | DDG § 5 Abs. 1 Nr. 2 | 🔴 falls fehlt |
| Handelsregisternr. + zuständiges Gericht (nur falls eingetragen) | DDG § 5 Abs. 1 Nr. 4 | 🔴 falls GmbH/AG ohne Angabe |
| USt-IdNr. oder Steuernummer (falls steuerpflichtig) | DDG § 5 Abs. 1 Nr. 6 | 🟡 falls nicht angegeben |
| Impressum maximal 2 Klicks erreichbar | BGH-Rechtsprechung | 🟡 falls tief vergraben |

Lookup: `gesetz_per_abkuerzung_abrufen("DDG")` → § 5

---

**2. Button-Lösung (BGB § 312j)**

| Prüfpunkt | Rechtsgrundlage | Klassifizierung |
|---|---|---|
| Bestellbutton mit gesetzlich konformem Wortlaut | BGB § 312j Abs. 3 | 🔴 falls falscher Text |
| Erlaubte Formulierungen (vollständige Liste in `references/shop-check-requirements.md`) | BGH I ZR 86/21 (2022) | 🔴 falls "Weiter", "Bestellung abschicken", "Jetzt anmelden" o.ä. |
| Unmittelbar vor Button: Zusammenfassung nach § 312j Abs. 2 | BGB § 312j Abs. 2 | 🔴 falls Zusammenfassung fehlt |
| Zusammenfassung enthält: wesentliche Waren-/Dienstleistungsmerkmale | BGB § 312j Abs. 2 Nr. 1 | 🔴 falls fehlt |
| Zusammenfassung enthält: Gesamtpreis inkl. aller Steuern und Abgaben | BGB § 312j Abs. 2 Nr. 2 | 🔴 falls fehlt |
| Checkout-Seite öffentlich zugänglich für Prüfung | — | ⚪ falls Login/Session erforderlich |

Lookup: `intelligente_rechtssuche("BGB § 312j Button-Lösung zahlungspflichtig bestellen")`

---

**3. Preisangaben (PAngV)**

| Prüfpunkt | Rechtsgrundlage | Klassifizierung |
|---|---|---|
| Gesamtpreis inkl. USt. ausgewiesen | PAngV § 3 Abs. 1 | 🔴 falls nur Netto-Preis |
| Hinweis auf USt. ("inkl. MwSt." oder "inkl. 19% MwSt.") | PAngV § 3 Abs. 1 | 🟡 falls kein MwSt.-Hinweis |
| Grundpreisangabe bei Waren nach Gewicht/Volumen/Länge/Fläche | PAngV § 4 | 🔴 falls fehlt (z.B. €/kg) |
| Streichpreise: niedrigster Preis der letzten 30 Tage als Referenz | PAngV § 11 | 🔴 falls UVP ohne 30-Tage-Basis |

Lookup: `intelligente_rechtssuche("PAngV Gesamtpreis Umsatzsteuer Grundpreis § 3")`

---

**4. Versandkosten und Lieferzeit (PAngV + BGB)**

| Prüfpunkt | Rechtsgrundlage | Klassifizierung |
|---|---|---|
| Versandkosten klar angegeben auf Produktseite | PAngV § 3 Abs. 2 | 🔴 falls erst im Checkout |
| Lieferzeit angegeben (z.B. "3–5 Werktage") | BGB § 308 Nr. 1 | 🟡 falls fehlt |
| "Kostenloser Versand" — Bedingungen klar kommuniziert (ab €X) | PAngV § 3 | 🟡 falls Schwellenwert unklar |
| Versandeinschränkungen (nur DE, nur EU etc.) kommuniziert | PAngV | 🟡 falls unklar |

---

**5. Widerrufsbelehrung (BGB §§ 355–357)**

| Prüfpunkt | Rechtsgrundlage | Klassifizierung |
|---|---|---|
| Widerrufsbelehrung vorhanden und erreichbar | BGB § 312g, § 355 | 🔴 falls fehlt |
| Widerrufsfrist: 14 Tage klar angegeben | BGB § 355 Abs. 2 | 🔴 falls falsche Frist oder fehlt |
| Fristbeginn korrekt (bei Waren: Erhalt; bei Dienstl.: Vertragsschluss) | BGB § 355 Abs. 2 Satz 2 | 🔴 falls falsch |
| Empfänger des Widerrufs angegeben (Name, Adresse, ggf. E-Mail) | BGB § 355 Abs. 1 | 🔴 falls fehlt |
| Rechtsfolgen erläutert (Rückgewähr innerhalb 14 Tage; Rücksendekosten) | BGB §§ 355, 357 | 🔴 falls fehlt |
| Ausnahmen vom Widerrufsrecht angegeben (z.B. verderbliche Waren, Digitales) | BGB § 312g Abs. 2 | 🟡 falls relevant aber fehlt |

Lookup: `intelligente_rechtssuche("BGB Widerrufsrecht Fernabsatz § 355 Widerrufsfrist 14 Tage")`

---

**6. Muster-Widerrufsformular (BGB § 356 Abs. 1, Anlage 2)**

| Prüfpunkt | Rechtsgrundlage | Klassifizierung |
|---|---|---|
| Muster-Widerrufsformular vorhanden (als Teil der Widerrufsbelehrung oder separat) | BGB § 356 Abs. 1 i.V.m. Anlage 2 | 🔴 falls fehlt |
| Formular enthält: Unternehmensanschrift als Empfänger | Anlage 2 zu § 356 BGB | 🔴 falls fehlt |
| Formular enthält: Erklärungsformel ("Hiermit widerrufe(n) ich/wir...") | Anlage 2 zu § 356 BGB | 🔴 falls fehlt |
| Formular enthält: Felder für Name, Anschrift, Datum des Vertrags, Datum | Anlage 2 zu § 356 BGB | 🟡 falls Felder fehlen |

Lookup: `intelligente_rechtssuche("BGB Anlage 2 § 356 Muster-Widerrufsformular")`

---

**7. AGB (BGB §§ 305–306)**

| Prüfpunkt | Rechtsgrundlage | Klassifizierung |
|---|---|---|
| AGB vorhanden und erreichbar | BGB § 305 | 🔴 falls fehlt |
| AGB in Kaufprozess einbezogen — Hinweis vor Bestellabschluss + Zustimmung | BGB § 305 Abs. 2 | 🔴 falls kein Hinweis oder Zustimmung fehlt |
| AGB für Kunden speicherbar und druckbar (kein reines Popup) | BGB § 312f Abs. 1 | 🔴 falls nicht speicherbar |
| AGB-Checkbox nicht vorausgefüllt (kein Pre-check) | BGB § 308 Nr. 5 — analog | 🟡 falls vorausgefüllt |

Lookup: `intelligente_rechtssuche("BGB § 305 AGB Einbeziehung Fernabsatz")`

---

### A3 — Gesetzestext nachschlagen

Für jeden 🔴 Befund den Gesetzestext via MCP einholen und wörtlich zitieren:

| Gesetz | MCP-Abfrage |
|---|---|
| DDG § 5 | `gesetz_per_abkuerzung_abrufen("DDG")` |
| BGB § 312j | `intelligente_rechtssuche("BGB § 312j Button-Lösung Bestellbutton")` |
| BGB §§ 355–357 | `intelligente_rechtssuche("BGB § 355 Widerrufsrecht Frist 14 Tage")` |
| BGB § 356 Anlage 2 | `intelligente_rechtssuche("BGB § 356 Muster-Widerrufsformular Anlage 2")` |
| PAngV § 3 | `intelligente_rechtssuche("PAngV § 3 Gesamtpreis Umsatzsteuer Versandkosten")` |
| BGB § 305 | `intelligente_rechtssuche("BGB § 305 Allgemeine Geschäftsbedingungen Einbeziehung")` |

Falls MCP nicht verfügbar: *(Gesetzestext aus Trainingsdaten — verifizieren auf gesetze-im-internet.de)*

### A4 — Ausgabeformat

```
## Shop-Check — [url]  |  [Datum]  |  Gesetzesquelle: [MCP/Trainingsdaten]

🔴 Verstöße — [N]

  [Bereich] [Prüfpunkt]
  Fundstelle: [URL / HTML-Beschreibung / "nicht gefunden"]
  Rechtsgrundlage: [Gesetz + §]
  Wortlaut: "[zitierter Gesetzestext aus MCP]"
  Behebung: [konkrete Maßnahme in einem Satz]

🟡 Risiken — [N]
  [Bereich] [Prüfpunkt] — [Empfehlung]

🟢 Konform — [N]
  [Bereich]: [Kurzbegründung]

⚪ Manuelle Prüfung erforderlich
  - Button-Lösung: Checkout-Seite nicht öffentlich zugänglich
  - AGB-Einbindung: Zustimmungs-Checkbox im Kaufprozess manuell prüfen
  [weitere nicht prüfbare Punkte]

Prüfbereiche-Übersicht:
  1. Impressum (DDG § 5):            [✓ / ⚠ / ✗]
  2. Button-Lösung (§ 312j BGB):     [✓ / ⚠ / ✗ / ⚪]
  3. Preisangaben (PAngV):           [✓ / ⚠ / ✗]
  4. Versandkosten/Lieferzeit:       [✓ / ⚠ / ✗]
  5. Widerrufsbelehrung (§§ 355 BGB):[✓ / ⚠ / ✗]
  6. Widerrufsformular (§ 356 BGB):  [✓ / ⚠ / ✗]
  7. AGB (§ 305 BGB):                [✓ / ⚠ / ✗]

Gesamtbewertung: [konform / mit Mängeln / kritische Verstöße]
Abmahnrisiko: [gering / mittel / hoch] — [Begründung]

Sofortmaßnahmen (🔴 nach Priorität):
1. [Konkrete Maßnahme — höchste Abmahngefahr]
2. [...]

*Dieser Bericht dient der technischen Orientierung und ersetzt keine Rechtsberatung.*
```

---

## Modus B — Code-Audit

Prüft die Checkout-Implementierung auf rechtskritische Anti-Patterns.

### B1 — Grep-Patterns

```bash
# Bestellbutton-Text (Button-Lösung § 312j BGB)
grep -rEi "(bestellung|checkout|order|cart|payment|buy|submit)" \
  --include="*.tsx" --include="*.jsx" --include="*.html" . | \
  grep -iE "(<button|type=\"submit\")"

# Widerrufsbelehrung / AGB — Links und Referenzen
grep -rEi "(widerruf|widerrufsbelehrung|agb|allgemeine.gesch|terms|cancellation|return.policy)" \
  --include="*.tsx" --include="*.jsx" --include="*.html" --include="*.ts" .

# Preisdarstellung — MwSt.-Kennzeichnung
grep -rEi "(price|preis|mwst|ust\.|tax|vat|incl\.|inkl\.)" \
  --include="*.tsx" --include="*.jsx" --include="*.ts" .

# Versandkosten
grep -rEi "(shipping|versand|lieferung|delivery|porto|frachtkosten)" \
  --include="*.tsx" --include="*.jsx" --include="*.ts" .

# AGB-Checkbox / Zustimmung vor Bestellung
grep -rEi "(agb|terms|checkbox.*agree|accept.*terms|zustimm|einverstanden)" \
  --include="*.tsx" --include="*.jsx" .

# Streichpreise — Referenzpreis-Logik
grep -rEi "(originalPrice|uvp|rrp|compareAtPrice|streichpreis|was_price|original_price)" \
  --include="*.tsx" --include="*.jsx" --include="*.ts" .
```

### B2 — Klassifizieren

Häufige Problemmuster:

| Code-Muster | Befund | Rechtsgrundlage |
|---|---|---|
| Button-Text: "Weiter", "Bestellen", "Bestellung abschicken", "Jetzt kaufen" (nicht qualifiziert) | 🔴 | BGB § 312j Abs. 3 |
| Kein AGB-Link oder Zustimmungs-Hinweis vor dem Bestellbutton | 🔴 | BGB § 305 Abs. 2 |
| Preis-Komponente ohne MwSt.-Hinweis | 🔴 | PAngV § 3 Abs. 1 |
| Kein Link zu Widerrufsbelehrung auf Checkout-Seite | 🔴 | BGB § 312g, § 312d |
| Versandkosten erst nach Adresseingabe sichtbar (ohne Hinweis davor) | 🟡 | PAngV § 3 Abs. 2 |
| Streichpreis-Logik ohne 30-Tage-Referenzpreis in DB | 🟡 | PAngV § 11 |
| AGB-Checkbox mit `defaultChecked` / `checked` | 🔴 | BGB § 308 Nr. 5 analog |

### B3 — Ausgabeformat

```
## Shop-Check Code-Audit — [Projekt]  |  [Datum]

🔴 [N] Verstöße  |  🟡 [N] Risiken  |  🟢 [N] Bestanden

🔴 [Datei:Zeile] [Problem]
  Rechtsgrundlage: [Gesetz + §]
  Wortlaut: "[Gesetzestext aus MCP falls verfügbar]"
  Fix: [konkreter Code-Vorschlag]

🟡 [Datei:Zeile] [Problem]
  Empfehlung: [Maßnahme]
```

---

## Modus C — Textbausteine

Generiert rechtskonforme deutsche Textbausteine für den Shop. Alle Pflichtinhalte werden live via MCP abgerufen — keine statischen Templates.

Unterstützte Typen:

| Typ | MCP-Abfrage | Ausgabe |
|---|---|---|
| `widerrufsbelehrung` | `gesetz_per_abkuerzung_abrufen("EGBGB")` → Anlage 1 zu Art. 246a § 1 Abs. 2 | Widerrufsbelehrung nach amtlichem Muster (Waren) |
| `widerrufsformular` | `gesetz_per_abkuerzung_abrufen("BGB")` → Anlage 2 zu § 356 | Muster-Widerrufsformular (amtlicher Text) |
| `impressum` | `gesetz_per_abkuerzung_abrufen("DDG")` → § 5 | Impressum-Bausteine mit allen Pflichtangaben |
| `button-texte` | `intelligente_rechtssuche("BGB § 312j Absatz 3 Bestellbutton zahlungspflichtig")` + `rechtsprechung_suchen("BGH I ZR 86/21 Button-Lösung")` | Erlaubte Formulierungen nach Gesetz + BGH |
| `agb-checkliste` | `gesetz_per_abkuerzung_abrufen("BGB")` → §§ 305–309 | AGB-Einbeziehungspflichten und verbotene Klauseln |

Ablauf für jeden Textbaustein:
1. MCP-Abfrage ausführen (oben angegeben) — Gesetzestext live abrufen
2. Pflichtinhalte direkt aus dem abgerufenen Text ableiten — nicht aus Trainingsdaten
3. Textbaustein generieren — jeden Pflichtinhalt mit Inline-Kommentar und exakter Rechtsgrundlage aus dem MCP-Abruf versehen
4. Falls MCP nicht verfügbar: explizit vermerken *(Gesetzestext aus Trainingsdaten — verifizieren auf gesetze-im-internet.de)*
5. Haftungsausschluss anhängen

Jeder generierte Text endet mit:
> *Dieser Text wurde mit KI-Unterstützung auf Basis des geltenden deutschen Rechts erstellt. Lassen Sie wichtige Shoprechtstexte von einem zugelassenen Rechtsanwalt prüfen. Kein Ersatz für Rechtsberatung.*

---

## Modus D — Vollprüfung

1. URL-Scan (Modus A) — alle sieben Bereiche
2. Falls Codebase-Pfad angegeben: Code-Audit (Modus B)
3. Konsolidierter Befundbericht — alle 🔴 Verstöße nach Abmahnrisiko sortiert
4. Angebot: "Soll ich rechtskonforme Textbausteine für die erkannten Lücken generieren? (`/shop-check draft widerrufsbelehrung`)"

---

## Ausgabestandards

- **Schweregrad:** 🔴 Verstoß (Abmahn-/Bußgeldrisiko — sofort beheben) / 🟡 Risiko (empfohlen) / 🟢 Konform / ⚪ Manuell prüfen
- **Abmahnrisiko:** Jeder 🔴-Befund öffnet Abmahnrisiko durch Mitbewerber und Verbände (IDO Verband, Verbraucherzentrale, wettbewerbsrechtliche Abmahnung nach UWG § 3a)
- **Gesetzeszitate:** Immer direkt aus MCP-Abfrage. Falls nicht verfügbar: *(Gesetzestext aus Trainingsdaten — verifizieren auf gesetze-im-internet.de)*
- **Haftungsausschluss:** Jeder Bericht endet mit — *Dieser Bericht dient der technischen Orientierung und ersetzt keine Rechtsberatung.*

