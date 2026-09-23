---
name: trmnl-tibber
description: "Use when the user asks about Tibber Germany retail electricity prices (hourly all-in EUR/kWh), cheapest appliance/EV windows, or pushing Tibber prices to a TRMNL e-ink Private Plugin webhook. Requires TIBBER_ACCESS_TOKEN. Not for OMIE, Ostrom, or other providers."
homepage: https://developer.tibber.com
metadata:
  openclaw:
    emoji: "⚡"
    primaryEnv: TIBBER_ACCESS_TOKEN
    requires:
      env:
        - TIBBER_ACCESS_TOKEN
      bins:
        - python3
---

# Tibber → TRMNL

## When to use

Use when the user asks about:
- Current or upcoming Tibber retail prices (Germany, hourly)
- Cheapest time to run a load
- Pushing prices to a **TRMNL** e-ink Private Plugin

## Setup

```bash
cp .env.example .env
# set TIBBER_ACCESS_TOKEN, optional TIBBER_HOME_ID, optional TRMNL_PLUGIN_UUID
```

Or `~/.config/tibber-energy/config.json` (same keys as [tibber-energy](https://github.com/pmagnomuller/tibber-energy)).

## Run

```bash
bash run.sh prices
bash run.sh trmnl
bash run.sh trmnl --push
```

## Notes

- Prices from `viewer.homes[].currentSubscription.priceInfo` (`total` = all-in EUR/kWh).
- Timestamps: `Europe/Berlin`. Resolution: hourly.
- Full TRMNL setup: [`trmnl/SETUP.md`](trmnl/SETUP.md).

## Safety

- Never commit `.env` with real tokens. Use GitHub secrets for CI.
