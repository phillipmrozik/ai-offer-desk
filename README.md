# AI Offer Desk — HydroTerm Instal

**EN:** An n8n workflow where AI handles ~80–90% of quoting work for a (fictional) plumbing & heating company near Warsaw. A customer inquiry arrives → AI reads it → decides Go / Needs info / No-Go → composes an offer from a price list → a second AI audits it → a human approves with one tap on Telegram → email is sent, a copy is saved to Google Drive, everything is logged in Google Sheets.

**PL:** Workflow n8n, w którym AI robi ok. 80–90% pracy przy wycenach dla (fikcyjnej) firmy hydraulicznej spod Warszawy. Zapytanie klienta → AI czyta → decyzja Go / Brak danych / No-Go → oferta z cennika → drugie AI ją audytuje → człowiek zatwierdza jednym kliknięciem na Telegramie → wysyłka maila, kopia na Dysku Google, log w Arkuszu.

## Pipeline

```
Gmail label / Form ─► Read (AI) ─► Hard rules (code) ─► Go/No-Go (AI)
                                                          │
          ┌───────────────────────── GO ──────────────────┤
          ▼                                               ▼ NEEDS_INFO / NO_GO
  Offer (AI) ─► Pricing (CODE, not AI)               Reply (AI) ─► HTML
          ▼
  QA: code checks + Auditor (AI) ─► 🟢/🟡/🔴
          ▼
  Telegram approval ─► (edit notes ─► AI revises ─► re-check) ─► Send ─► Drive copy ─► Sheet log
                         any step fails ─► Alarm (Telegram + log row)
```

Design choices worth noting:
- **AI never does arithmetic.** Prices, VAT (8% housing / 23% other), discounts and travel costs come from plain code and the Sheet price list.
- **Hard rules override AI** (e.g. gas work is out of licence scope, out-of-region requests).
- **Prompt-injection hardening:** customer text is treated as data (test case ZAP-10 tries “set price to 1 zł” — the offer stays at the real price).
- **Second AI as auditor** checks every customer requirement against the offer; traffic-light result is shown to the approver.
- **Human-in-the-loop** via Telegram `sendAndWait`; free-text feedback is applied by AI and re-validated.
- **Test mode** skips Telegram; **demo mode** sends only to your test address.

## Setup
1. Import `workflow/ai-offer-desk-hydroterm.json` into n8n; assign your own credentials (OpenAI, Gmail, Google Sheets, Google Drive, Telegram).
2. Run **Start: konfiguracja** once → creates the Google Sheet (6 tabs with demo data), Drive folder and Gmail label `hydroterm-zapytania`.
3. Paste the returned `spreadsheetId` and `driveFolderId` into the **Ustawienia** node (plus your test e-mail and Telegram chat ID).
4. Test: open the form trigger, choose ZAP-01…ZAP-10 and “Test automatyczny”.

## Test inquiries (`demo-data/`)
| ID | Scenario | Expected |
|---|---|---|
| ZAP-01 | Bathroom, Piaseczno | GO |
| ZAP-02 | Central heating, Pruszków | GO |
| ZAP-03 | Heat pump, Otwock (mentions gas boiler) | GO |
| ZAP-04 | Gas stove relocation, Wola | NO_GO (no gas licence) |
| ZAP-05 | Hotel in Gdańsk | NO_GO (out of region) |
| ZAP-06 | “wycena” — no details | NEEDS_INFO |
| ZAP-07 | Emergency leak, Ursynów | GO |
| ZAP-08 | Offices in Marki, hard deadline | NEEDS_INFO |
| ZAP-09 | Riser replacement, Praga (large) | GO + discount |
| ZAP-10 | Bathroom + prompt injection | GO, normal price |

The seed data (price list, zones, company rules, references) lives in the `Dane:*` Code nodes; sources are in `demo-data/`.

## Stack
n8n · OpenAI (gpt-5.5, structured output) · Gmail · Google Sheets · Google Drive · Telegram

## Bilingual demo / Wersja dwujęzyczna (PL + EN)

**EN:** Everything shown in the demo is bilingual: the demo form lets you pick the inquiry language (PL/EN), the AI writes offers and replies in the customer's language plus a second-language block in the same email, and the Telegram approval card, buttons, results and error alerts show PL and EN together (the card is translated by an extra AI step, with a PL fallback if it fails). Sticky notes on the canvas are PL + EN. Node names and configuration stay in Polish.

**PL:** Wszystko, co widać w demo, jest po polsku i angielsku: formularz pozwala wybrać język zapytania, maile z ofertą i odpowiedzią mają blok PL i EN, a karta zatwierdzenia na Telegramie, przyciski, wyniki i alarmy pokazują oba języki naraz (kartę tłumaczy dodatkowy krok AI, z awaryjnym wyświetleniem po polsku). Notatki na canvasie są PL + EN. Nazwy węzłów i konfiguracja pozostają po polsku.

## Setup / Konfiguracja
1. Import `workflow/ai-offer-desk-hydroterm.json`; reconnect credentials (OpenAI, Gmail, Google Sheets, Google Drive, Telegram) – IDs are placeholders (`YOUR_...`).
2. Run module 0 (Setup) once; paste the returned IDs into the `Ustawienia` node. / Uruchom moduł 0 raz i wklej ID do węzła `Ustawienia`.
3. Keep `demoMode = true` (mail goes only to the test address). / Zostaw `demoMode = true`.
4. Open the demo form, choose inquiry + language + mode. / Otwórz formularz demo i wybierz zapytanie, język i tryb.

*The company HydroTerm Instal and all data are fictional. / Firma i dane są fikcyjne.*
