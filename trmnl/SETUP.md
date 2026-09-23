# TRMNL Private Plugin — Tibber Germany hourly prices

Push Tibber all-in retail prices (EUR/kWh `total`) to your TRMNL e-ink display every 15 minutes.

## One-time setup

### 1. Create a Private Plugin

1. In [TRMNL](https://trmnl.com) go to **Plugins** → **Private Plugin**.
2. Set **Strategy** to **Webhook**.
3. Save the plugin so a **Webhook URL** / UUID appears.
4. Copy the UUID from `https://trmnl.com/api/custom_plugins/<UUID>`.

### 2. Paste markup

Open the Markup editor and paste:

| Size | File |
|------|------|
| Full | [`src/full.liquid`](src/full.liquid) |
| Half vertical | [`src/half_vertical.liquid`](src/half_vertical.liquid) |
| Quadrant | [`src/quadrant.liquid`](src/quadrant.liquid) |

Title bars say **Tibber**.

### 3. Configure credentials / CI

Local dry-run (prints JSON, no push):

```bash
bash run.sh trmnl
```

Push once:

```bash
export TIBBER_ACCESS_TOKEN="..."
export TRMNL_PLUGIN_UUID="your-uuid-here"
# optional: export TIBBER_HOME_ID="..."
bash run.sh trmnl --push
```

GitHub Actions secrets on this repo:

| Name | Required |
|------|----------|
| `TIBBER_ACCESS_TOKEN` | yes |
| `TRMNL_PLUGIN_UUID` | yes |
| `TIBBER_HOME_ID` | no |

The workflow [`.github/workflows/trmnl-tibber.yml`](../.github/workflows/trmnl-tibber.yml) runs every 15 minutes and on manual dispatch.

### 4. Playlist

Add the plugin to your device playlist. A refresh interval of **15–30 minutes** is enough.

## Payload fields

| Field | Meaning |
|-------|---------|
| `current.price_label` | All-in EUR/kWh (`total`) for the active hour |
| `current.slot_label` | e.g. `13:00–14:00` (Europe/Berlin) |
| `today_min_label` / `today_max_label` / `today_avg_label` | Today’s hourly stats |
| `cheapest_next.label` | Next cheapest 1h block (default) |
| `upcoming.t` / `upcoming.p` / `upcoming.bars` | Sparkline labels, cents/kWh, 0–10 bar heights |
| `tomorrow_ready` | `true` once Tibber returns tomorrow’s hours |
| `updated_label` | Last push time (Berlin) |

Retail all-in (energy + tax as returned by Tibber) — not wholesale spot alone.
