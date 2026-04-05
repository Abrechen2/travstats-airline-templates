# TravStats Airline Email Templates

Community-maintained parser templates for [TravStats](https://github.com/Abrechen2/TravStats).

Templates are automatically synced daily into TravStats instances.

## Structure

```
templates/
  index.json        ← registry of all templates with versions
  LH.json           ← Lufthansa (new format, 2025)
  LH-old.json       ← Lufthansa (Buchungsdetails format)
  EW.json           ← Eurowings
  FR.json           ← Ryanair
  LX.json           ← Swiss
  OS.json           ← Austrian
  SN.json           ← Brussels Airlines
  U2.json           ← easyJet
  W6.json           ← Wizz Air
```

## Template Format

Each template is a JSON file with this structure:

```json
{
  "airline": "Airline Name",
  "iata": "XX",
  "version": "YYYY-MM",
  "from": ["@airline.com"],
  "subject": ["Booking Confirmation", "Buchungsbestätigung"],
  "selectors": {
    "flightNumber": ".css-selector",
    "pnr": ".css-selector",
    "departureCode": ".css-selector",
    "arrivalCode": ".css-selector",
    "departureTime": ".css-selector",
    "arrivalTime": ".css-selector"
  },
  "textPatterns": {
    "flightNumber": ["Flight(?:\\s*No\\.?)?:\\s*([A-Z]{2}\\s?\\d{1,4})"],
    "pnr": ["Booking\\s*Reference:\\s*([A-Z0-9]{5,8})"],
    "departureCode": ["From:[^(\\n]*\\(([A-Z]{3})\\)"],
    "arrivalCode": ["To:[^(\\n]*\\(([A-Z]{3})\\)"],
    "departureTime": ["Departure:\\s*(\\d{1,2}\\s+\\w+\\s+\\d{4})[\\s\\S]{0,80}?Time:\\s*(\\d{1,2}:\\d{2})"]
  },
  "transforms": {
    "flightNumber": "removeSpaces",
    "pnr": "uppercase",
    "departureCode": "uppercase",
    "arrivalCode": "uppercase",
    "departureTime": "parseIso",
    "arrivalTime": "parseIso"
  },
  "testCases": [
    {
      "input": "Flight: LH 100\nFrom: Munich (MUC)\nTo: London (LHR)",
      "expected": {
        "flightNumber": "LH100",
        "departureCode": "MUC",
        "arrivalCode": "LHR"
      }
    }
  ]
}
```

### Fields

| Field | Description |
|-------|-------------|
| `airline` | Full airline name |
| `iata` | Airline IATA code (unique key) |
| `version` | Version string `YYYY-MM` — bump when changing patterns |
| `from` | Sender domain(s) to match (e.g. `@lufthansa.com`) |
| `subject` | Subject line keywords to match |
| `selectors` | CSS selectors for HTML emails (cheerio) |
| `textPatterns` | Regex patterns for plain-text emails (fallback) |
| `transforms` | Value transforms: `trim`, `uppercase`, `lowercase`, `removeSpaces`, `stripNonAlpha`, `extractIata`, `extractFlightNumber`, `parseIso` |
| `testCases` | Test cases for validation |

### textPatterns

Each key maps to an array of regex strings tried in order. The **first capture group** is used as the value. If two capture groups are present, they are joined as `{group1}T{group2}` (for date+time combinations).

### transforms — parseIso

Converts human-readable dates to ISO 8601:
- `"18 Sep 2025T07:25"` → `"2025-09-18T07:25"`
- `"13. Dezember 2024 16:05"` → `"2024-12-13T16:05"`

## Contributing

1. Fork this repo
2. Add your template in `templates/XX.json`
3. Update `templates/index.json` — bump the top-level `version` and add your entry
4. Open a PR with a test case from a real (anonymized) email

**Please anonymize test cases** — replace real booking codes, names, and ticket numbers with fictional ones.
