# gdpr-dsgvo-audit

**A Claude Code skill for German DSGVO / GDPR compliance** — code audits, live URL scans, data flow mapping, and policy drafting, grounded in actual German federal law via the [rechtsinformationen.bund.de](https://www.rechtsinformationen.bund.de/) MCP.

**Ein Claude Code Skill für DSGVO / GDPR-Compliance** — Code-Audits, Live-URL-Scans, Datenfluss-Mapping und Dokumentenentwürfe, verankert in echten deutschen Gesetzestexten via [rechtsinformationen.bund.de](https://www.rechtsinformationen.bund.de/) MCP.

> ⚠️ **Not legal advice / Keine Rechtsberatung.** This skill helps developers identify and document DSGVO-relevant patterns. Always have outputs reviewed by a qualified Datenschutzbeauftragter or attorney. / Dieser Skill hilft Entwicklern, DSGVO-relevante Muster zu identifizieren und zu dokumentieren. Outputs sollten stets von einem qualifizierten Datenschutzbeauftragten oder Rechtsanwalt geprüft werden.

---

## English

### What it does

| Command | What happens |
|---|---|
| `/legal-audit audit <path>` | Scans your codebase — finds third-party scripts, PII collection, consent gaps, missing Impressum. Severity: 🔴 Hoch / 🟡 Mittel / 🟢 Niedrig |
| `/legal-audit scan <url>` | Fetches the live site — detects undisclosed third-party scripts, audits consent banner correctness, checks required pages |
| `/legal-audit flow <path>` | Maps all PII flows → generates an Art. 30 VVT (Verzeichnis von Verarbeitungstätigkeiten) and DPA checklist |
| `/legal-audit draft <type>` | Generates bilingual (DE + EN) sections: `privacy-section`, `privacy-page`, `impressum`, `cookie-banner`, `vvt`, `art13-notice` |
| `/legal-audit` | Full suite: audit + scan + flow, then offers to draft fixes |

**Auto-triggers** on natural-language legal questions — say "is our privacy policy compliant?" or "do we need a DPA for Stripe?" and the skill activates automatically.

### Law coverage

- **DSGVO** — Art. 5, 6, 7, 13, 25, 28, 30, 32, 33, 46, 83
- **TTDSG** — § 25(1) device access consent, § 25(2) strictly necessary exemption
- **BDSG** — § 26 employee data, § 38 DPO threshold
- **TMG / DDG** — § 5 Impressum requirements
- **Key case law** — LG München I 2022 (Google Fonts), Planet49 (CJEU 2019), Schrems II (CJEU 2020), BGH Cookie Banner 2020

### The differentiator: live law lookups

When the `rechtsinformationen-bund-de` MCP is configured, the skill pulls **live statutory text** from the official German federal law portal before citing any article. Without it the skill still works, but falls back to training knowledge and flags it.

### Installation

**1. Install the skill**

```bash
git clone https://github.com/waldo-van-der-code/gdpr-dsgvo-audit.git ~/.claude/skills/legal-audit
```

The skill is immediately available as `/legal-audit` in any Claude Code session.

**2. (Optional but recommended) Install the live law MCP**

```bash
git clone --depth 1 https://github.com/wolfgangihloff/rechtsinformationen-bund-de-mcp.git \
  ~/.claude/mcp-servers/rechtsinformationen-bund-de
cd ~/.claude/mcp-servers/rechtsinformationen-bund-de
npm install && npm run build
```

Add to `~/.mcp.json`:

```json
{
  "mcpServers": {
    "rechtsinformationen-bund-de": {
      "type": "stdio",
      "command": "node",
      "args": ["~/.claude/mcp-servers/rechtsinformationen-bund-de/dist/index.js"]
    }
  }
}
```

Restart Claude Code / Claude Desktop to activate.

### Usage examples

```
/legal-audit audit ./my-nextjs-app
/legal-audit scan https://example.de
/legal-audit draft privacy-section "Resend (transactional email)"
/legal-audit draft impressum
/legal-audit draft cookie-banner

# Natural language — skill auto-triggers:
"Is our cookie banner valid under TTDSG?"
"Do we need a DPA for Vercel?"
"Our site loads Google Fonts from CDN — is that a problem?"
```

### Seeking expert input 🙏

Built by a developer, not a lawyer. Looking for:

- [ ] **Datenschutzbeauftragte / privacy lawyers** — are the article citations accurate? Gaps in the violation-to-article mapping?
- [ ] **DPA practitioners** — what patterns actually attract supervisory authority attention?
- [ ] **Austrian DSG coverage** — differences from DSGVO need proper treatment
- [ ] **Swiss nDSG coverage** — in force since Sept 2023, meaningfully different from DSGVO
- [ ] **DPIA screening** — Art. 35 and published DPA lists of high-risk processing

Open an issue or PR. Legal professionals especially welcome — please note your background so contributions can be weighted appropriately.

---

## Deutsch

### Was der Skill macht

| Befehl | Funktion |
|---|---|
| `/legal-audit audit <pfad>` | Scannt den Quellcode — findet Drittanbieter-Skripte, PII-Verarbeitung, fehlende Einwilligungs-Gates, fehlendes Impressum. Schweregrad: 🔴 Hoch / 🟡 Mittel / 🟢 Niedrig |
| `/legal-audit scan <url>` | Ruft die Live-Website ab — erkennt nicht deklarierte Drittanbieter, prüft Cookie-Banner-Korrektheit, prüft Pflichtseiten |
| `/legal-audit flow <pfad>` | Kartiert alle PII-Datenflüsse → erstellt Art. 30-VVT und AV-Vertrag-Checkliste |
| `/legal-audit draft <typ>` | Generiert zweisprachige (DE + EN) Abschnitte: `privacy-section`, `privacy-page`, `impressum`, `cookie-banner`, `vvt`, `art13-notice` |
| `/legal-audit` | Vollständige Suite: Audit + Scan + Datenfluss, dann Angebot zur Dokumentenerstellung |

**Automatische Aktivierung** bei datenschutzrechtlichen Fragen — einfach fragen: „Ist unsere Datenschutzerklärung korrekt?" oder „Brauchen wir einen AV-Vertrag mit Stripe?" und der Skill wird aktiv.

### Gesetzliche Abdeckung

- **DSGVO** — Art. 5, 6, 7, 13, 25, 28, 30, 32, 33, 46, 83
- **TTDSG** — § 25 Abs. 1 Einwilligung für Gerätezugriff, § 25 Abs. 2 technisch notwendige Ausnahme
- **BDSG** — § 26 Beschäftigtendaten, § 38 Datenschutzbeauftragter-Schwelle
- **TMG / DDG** — § 5 Impressumspflicht
- **Wichtige Rechtsprechung** — LG München I 2022 (Google Fonts), Planet49 (EuGH 2019), Schrems II (EuGH 2020), BGH Cookie-Banner 2020

### Das Alleinstellungsmerkmal: echte Gesetzestexte

Mit dem `rechtsinformationen-bund-de` MCP ruft der Skill **live den aktuellen Gesetzestext** vom offiziellen deutschen Bundesrechtsportal ab, bevor er einen Artikel zitiert. Ohne MCP funktioniert der Skill trotzdem — greift aber auf Trainingswissen zurück und kennzeichnet dies.

### Installation

**1. Skill installieren**

```bash
git clone https://github.com/waldo-van-der-code/gdpr-dsgvo-audit.git ~/.claude/skills/legal-audit
```

Der Skill ist sofort als `/legal-audit` in jeder Claude Code-Sitzung verfügbar.

**2. (Optional, aber empfohlen) Live-Gesetzes-MCP installieren**

```bash
git clone --depth 1 https://github.com/wolfgangihloff/rechtsinformationen-bund-de-mcp.git \
  ~/.claude/mcp-servers/rechtsinformationen-bund-de
cd ~/.claude/mcp-servers/rechtsinformationen-bund-de
npm install && npm run build
```

In `~/.mcp.json` eintragen:

```json
{
  "mcpServers": {
    "rechtsinformationen-bund-de": {
      "type": "stdio",
      "command": "node",
      "args": ["~/.claude/mcp-servers/rechtsinformationen-bund-de/dist/index.js"]
    }
  }
}
```

Claude Code / Claude Desktop neu starten.

### Verwendungsbeispiele

```
/legal-audit audit ./mein-nextjs-projekt
/legal-audit scan https://example.de
/legal-audit draft privacy-section "Resend (transaktionale E-Mails)"
/legal-audit draft impressum
/legal-audit draft cookie-banner

# Natürlichsprachige Fragen — Skill aktiviert sich automatisch:
"Ist unser Cookie-Banner nach TTDSG § 25 korrekt?"
"Brauchen wir einen AV-Vertrag mit Vercel?"
"Wir laden Google Fonts von Googles CDN — ist das ein Problem?"
```

### Wir suchen Experten-Feedback 🙏

Dieser Skill wurde von einem Entwickler gebaut, nicht von einem Juristen. Wir suchen:

- [ ] **Datenschutzbeauftragte / Datenschutzanwälte** — sind die Artikel-Zitate korrekt? Gibt es Lücken im Verstoß-zu-Artikel-Mapping?
- [ ] **Behördenerfahrung** — welche Muster ziehen tatsächlich Aufmerksamkeit von BlnBDI, BayLDA & Co. auf sich?
- [ ] **Österreichisches DSG** — Unterschiede zur DSGVO brauchen korrekte Abdeckung
- [ ] **Schweizer nDSG** — seit September 2023 in Kraft, weicht von der DSGVO ab
- [ ] **DSFA-Screening** — Art. 35 DSGVO und Muss-Listen der Datenschutzbehörden

Issues und PRs willkommen. Rechtsprofis besonders willkommen — bitte Hintergrund angeben, damit Beiträge entsprechend gewichtet werden können.

---

## Project structure / Projektstruktur

```
gdpr-dsgvo-audit/
├── SKILL.md                        Core skill — routing, modes A–E
└── references/
    ├── law-reference.md            DSGVO/TTDSG/BDSG article table, case law, DPA contacts by Bundesland
    ├── audit-patterns.md           Grep patterns, domain classification, consent banner checklist
    ├── data-flow.md                Art. 30 VVT template, processor DPA URLs, transfer matrix
    └── document-templates.md       Art. 13 checklist, Impressum, cookie banner, rights section templates
```

## Inspiration

Built by studying and combining:

- [zubair-trabzada/ai-legal-claude](https://github.com/zubair-trabzada/ai-legal-claude) — URL scan pattern, parallel subagent approach
- [Sushegaad/Claude-Skills-Governance-Risk-and-Compliance](https://github.com/Sushegaad/Claude-Skills-Governance-Risk-and-Compliance) — code audit pattern, severity grading, document drafting
- [wolfgangihloff/rechtsinformationen-bund-de-mcp](https://github.com/wolfgangihloff/rechtsinformationen-bund-de-mcp) — live German federal law lookup

## License

MIT — see [LICENSE](LICENSE).
