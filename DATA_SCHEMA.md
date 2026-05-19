# CCF KPI Dashboard — Data Schema (v1.0)

This document defines the JSON shape that the CCF dashboard consumes. Power Automate (or any other ETL) must produce a `data.json` file matching this shape, hosted alongside the dashboard's `index.html`.

## File location

- **Filename**: `data.json`
- **Location**: same folder as `index.html` (dashboard fetches `./data.json` with a no-cache header)
- **Format**: UTF-8 encoded JSON, no BOM
- **Refresh**: re-write the file whenever the source workbook is updated

## Top-level structure

```json
{
  "client": { ... },
  "provider": { ... },
  "reportingYear": 2026,
  "lastUpdated": "2026-05-19T09:00:00Z",
  "schemaVersion": "1.0",
  "weeksMeta": [ ... ],
  "weeks": [ ... ]
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `client` | object | Yes | Client identity and branding |
| `provider` | object | Yes | PROMAN identity and branding |
| `reportingYear` | integer | Yes | Calendar year the data covers (e.g. `2026`) |
| `lastUpdated` | string (ISO 8601) | Yes | Timestamp of the most recent data refresh |
| `schemaVersion` | string | Yes | Schema version. Currently `"1.0"` |
| `weeksMeta` | array | Yes | Calendar metadata for every ISO week in the year |
| `weeks` | array | Yes | The actual KPI values, one entry per populated week |

## `client` object

```json
{
  "code": "CCF",
  "name": "Cranswick Continental Foods",
  "logoUrl": "https://promank13.azurewebsites.net/.../cranswick-continental-transparent.png"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `code` | string | Yes | Short client identifier, used in URLs and folder names |
| `name` | string | Yes | Full client name shown in the dashboard title |
| `logoUrl` | string | No | URL to the client logo (PNG, transparent background recommended) |

## `provider` object

```json
{
  "name": "PROMAN UK",
  "logoUrl": "https://proman-uk.com/img/logo.png"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | Provider name (always "PROMAN UK" for this dashboard) |
| `logoUrl` | string | No | URL to the PROMAN logo |

## `weeksMeta` array

One entry per ISO week in the reporting year. Generate all 52 or 53 weeks upfront, even if some are not yet populated.

```json
[
  {
    "w": 1,
    "mon": "2025-12-29",
    "sun": "2026-01-04",
    "mon_short": "29 Dec",
    "sun_short": "04 Jan",
    "month": 1
  },
  ...
]
```

| Field | Type | Required | Description |
|---|---|---|---|
| `w` | integer | Yes | ISO week number (1–52 or 1–53) |
| `mon` | string (YYYY-MM-DD) | Yes | Date of the Monday starting the week |
| `sun` | string (YYYY-MM-DD) | Yes | Date of the Sunday ending the week |
| `mon_short` | string | Yes | Short display label for Monday (e.g. "29 Dec") |
| `sun_short` | string | Yes | Short display label for Sunday (e.g. "04 Jan") |
| `month` | integer | Yes | Month the week belongs to (1–12). Use the month of the Sunday — this matches the workbook's week-to-month rule |

## `weeks` array

The dashboard accepts **two shapes** for the `weeks` array:

### Shape A — Nested (recommended for hand-written or schema-validated JSON)

One entry per week that has at least one data point. Weeks with no data should be omitted entirely.

```json
[
  {
    "week": 16,
    "values": {
      "0": 101,
      "1": 8227.04,
      "3": 61,
      "4": 40,
      "29": "No issues this week"
    }
  }
]
```

### Shape B — Flat (the shape Power Automate produces)

One entry per (week, KPI) pair. Easier to build from Excel table data because no grouping is required. The dashboard groups these into the nested shape on load.

```json
[
  { "week": 16, "kpi_index": 0,  "value_num": 101,     "value_text": "" },
  { "week": 16, "kpi_index": 1,  "value_num": 8227.04, "value_text": "" },
  { "week": 16, "kpi_index": 29, "value_num": "",      "value_text": "No issues this week" }
]
```

In Shape B:
- `value_num` is used for numeric KPIs (most of them). For text KPIs leave it as empty string.
- `value_text` is used for text KPIs (indices 20, 21, 29). For numeric KPIs leave it as empty string.
- Records where both `value_num` and `value_text` are empty should be filtered out at the source — they will be ignored by the dashboard if they reach it.

### Shape A field definitions

| Field | Type | Required | Description |
|---|---|---|---|
| `week` | integer | Yes | ISO week number, must match a `w` in `weeksMeta` |
| `values` | object | Yes | Map of KPI index (as string key) → value |

### KPI indices (used by both shapes)

The KPI index is the row position on the workbook's Weekly Input sheet, starting from 0. The mapping is:

| Index | KPI | Type | Aggregation |
|---:|---|---|---|
| 0 | Number of Temporary Workers | integer | Avg across weeks |
| 1 | Total Hours Worked | number | Sum across weeks |
| 2 | *(BANNER — Breakdown of Temporary Workers — not a data row)* | — | — |
| 3 | Sex: Male | integer | Avg |
| 4 | Sex: Female | integer | Avg |
| 5 | Location: Within 10 miles of site | integer | Avg |
| 6 | Age Groups: 16 - 18 | integer | Avg |
| 7 | Age Groups: 19 - 30 | integer | Avg |
| 8 | Age Groups: 31 - 40 | integer | Avg |
| 9 | Age Groups: 41 - 50 | integer | Avg |
| 10 | Age Groups: 51 - 60 | integer | Avg |
| 11 | Age Groups: 61+ | integer | Avg |
| 12 | Nationality: British | integer | Avg |
| 13 | Nationality: Polish | integer | Avg |
| 14 | Nationality: India | integer | Avg |
| 15 | Nationality: Romanian | integer | Avg |
| 16 | Nationality: Nigerian | integer | Avg |
| 17 | Nationality: Pakistan | integer | Avg |
| 18 | Nationality: Hong Kong (China) | integer | Avg |
| 19 | Nationality: Total Other | integer | Avg |
| 20 | Nationality: Other (define) | string | Concat by week |
| 21 | Ethnicity: Detail | string | Concat by week |
| 22 | Disabilities: Registered disabilities (managers) | integer | Avg |
| 23 | Health & Safety: Accidents reported | integer | Sum |
| 24 | Health & Safety: Inductions conducted | integer | Sum |
| 25 | Fulfilment: Shifts scheduled by CCF Planning | integer | Sum |
| 26 | Fulfilment: Shifts requested by Production | integer | Sum |
| 27 | Fulfilment: Shifts delivered by agency | integer | Sum |
| 28 | Fulfilment: Workers retained by CCF | integer | Avg |
| 29 | Issues raised: Concerns / feedback | string | Concat by week |

**Notes on the values object:**
- Keys are strings (JSON object keys are always strings), e.g. `"0"` not `0`
- Empty values: omit the key entirely. Do not use `null`, `""`, or `0` to mean "no data" — `0` is a valid value (no accidents reported, for example)
- Text fields (indices 20, 21, 29) are weekly raw text; the dashboard handles concatenation across weeks for monthly/YTD views
- Numeric fields are stored as integers where possible, except `Total Hours Worked` (index 1) which is a number with up to 2 decimal places

## Mapping from the workbook

The source of truth is the **Weekly Input** sheet of `CCF_KPI_Input.xlsx`. Each KPI is one row, and each ISO week is one column.

- **KPI rows start at row 7** of the Weekly Input sheet
- **KPI index 0** corresponds to row 7, **index 1** to row 8, and so on
- **Row 9 (KPI index 2)** is a banner row, not a data row — skip it
- **Week columns start at column E** (W01) and run through to column BE (W53)
- **Week W is in column** `(4 + W)` — so W01 = column E, W16 = column T, W53 = column BE
- **Hidden helper rows** (row 4 and row 38) on the Weekly Input sheet contain the month number and week number for each column — these are for the workbook's formulas and should not be exported

## Validation

A consumer (the dashboard) should treat the data defensively:
- Missing `weeks` array → show "no data" state
- Missing values for individual KPIs → display as "—" rather than 0
- Out-of-range KPI indices → ignore silently
- `lastUpdated` missing or unparseable → fall back to the latest populated week's Sunday date

## Versioning

If the schema needs to change (new KPI added, structure modified), bump `schemaVersion` and update this document. The dashboard should read `schemaVersion` on load and warn if it doesn't recognise the version.
