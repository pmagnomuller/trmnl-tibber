# trmnl-tibber

Tibber Germany retail electricity prices (hourly, all-in EUR/kWh) on a [TRMNL](https://trmnl.com) e-ink display.

A GitHub Actions cron runs every 15 minutes. The script authenticates to the Tibber GraphQL API, reads `priceInfo.today` / `tomorrow` (total = energy + tax), computes the current hour, today's min/avg/max, the cheapest upcoming 1h window and a sparkline, and POSTs a compact JSON payload to a TRMNL Private Plugin webhook. TRMNL renders it with the Liquid markup in this repo.

Modeled on [trmnl-omie](https://github.com/pmagnomuller/trmnl-omie); reuses auth + fetch from [tibber-energy](https://github.com/pmagnomuller/tibber-energy).

---

## Setup

### 1. Create a Private Plugin

1. In [TRMNL](https://trmnl.com) go to **Plugins** → **Private Plugin** → **Add**.
2. Set **Strategy** to **Webhook**.
3. Name it **Tibber**, save. Copy the UUID from  
   `https://trmnl.com/api/custom_plugins/<UUID>`.

### 2. Paste markup (or connect GitHub)

| Size | File |
|------|------|
| Full | [`trmnl/src/full.liquid`](trmnl/src/full.liquid) |
| Half vertical | [`trmnl/src/half_vertical.liquid`](trmnl/src/half_vertical.liquid) |
| Quadrant | [`trmnl/src/quadrant.liquid`](trmnl/src/quadrant.liquid) |

Or connect the plugin to this repo (folder `trmnl`) so `trmnl/src/*` syncs both ways.

### 3. Secrets

| Secret | Required | Source |
|--------|----------|--------|
| `TIBBER_ACCESS_TOKEN` | yes | [developer.tibber.com](https://developer.tibber.com) personal access token |
| `TIBBER_HOME_ID` | no | only if the account has multiple homes |
| `TRMNL_PLUGIN_UUID` | yes | UUID from the webhook URL |

Locally:

```bash
cp .env.example .env
# edit .env — never commit it
bash run.sh prices
bash run.sh trmnl          # dry-run JSON
bash run.sh trmnl --push   # push once
```

On GitHub:

```bash
gh secret set TIBBER_ACCESS_TOKEN -R pmagnomuller/trmnl-tibber
# optional:
# gh secret set TIBBER_HOME_ID -R pmagnomuller/trmnl-tibber
gh secret set TRMNL_PLUGIN_UUID -R pmagnomuller/trmnl-tibber
gh workflow run trmnl-tibber.yml -R pmagnomuller/trmnl-tibber
```

### 4. Playlist

Add the plugin to your device playlist. 15–30 min refresh is enough; Tibber day-ahead prices change a few times a day, but the “Now” hour and cheapest-next labels update each push.

Field-level payload reference: [`trmnl/SETUP.md`](trmnl/SETUP.md).

---

## Payload

Same shape as `trmnl-omie`, adapted for hourly all-in retail:

```json
{
  "area": "DE",
  "area_label": "Home nickname or city",
  "updated_at": "2026-09-23T10:30:00+02:00",
  "updated_label": "10:30",
  "current": {
    "price_eur_kwh": 0.2841,
    "price_cents_kwh": 28,
    "price_label": "0.2841",
    "starts_at": "...",
    "ends_at": "...",
    "slot_label": "10:00–11:00",
    "level": "NORMAL"
  },
  "today": { "min": 0.19, "max": 0.35, "avg": 0.26 },
  "today_min_label": "0.1900",
  "today_max_label": "0.3500",
  "today_avg_label": "0.2600",
  "cheapest_next": {
    "label": "13:00–14:00",
    "avg_eur_kwh": 0.2012,
    "avg_cents_kwh": 20,
    "hours": 1
  },
  "upcoming": { "t": ["10:00", "..."], "p": [28, "..."], "bars": [7, "..."], "count": 24 },
  "tomorrow_ready": true
}
```

- Prices are Tibber **`total`** (all-in EUR/kWh).
- Timestamps use **`Europe/Berlin`**.
- Resolution is **hourly** (not 15-min). Soft size guard shrinks `upcoming` until compact JSON ≤ 2 KB.

---

## Local CLI

```bash
bash run.sh prices --hours 36
bash run.sh optimize --duration-hours 2
bash run.sh trmnl
bash run.sh trmnl --push
```

Config precedence: env / `.env` → `~/.config/tibber-energy/config.json`.

---

## Repo layout

```
.
├── tibber_energy.py              # fetch, compute, CLI, TRMNL push
├── run.sh
├── requirements.txt              # stdlib only
├── .env.example
├── config.json.example
├── SKILL.md
├── trmnl/
│   ├── SETUP.md
│   └── src/                      # full / half_vertical / quadrant + settings.yml
└── .github/workflows/
    └── trmnl-tibber.yml          # */15 cron + workflow_dispatch
```

## Safety

- Never commit `.env` or real tokens. Use `.env.example` and GitHub Actions secrets only.
- Related: [tibber-energy](https://github.com/pmagnomuller/tibber-energy), [trmnl-omie](https://github.com/pmagnomuller/trmnl-omie).
