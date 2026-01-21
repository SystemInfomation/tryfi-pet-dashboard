<p align="center">
  <img src="https://img.shields.io/badge/PYTRYFI-Dog%20Collar%20API-22c55e?style=for-the-badge&labelColor=0f172a" alt="pytryfi" />
</p>

<h1 align="center">🐕 pytryfi</h1>
<h3 align="center">Python + Next.js • TryFi Collar API • Real-Time Pet Tracking</h3>

<p align="center">
  <strong>Location. Activity. Sleep. Bases. LED. Lost Dog Mode.</strong> — All from the TryFi GraphQL API.
</p>

---

<p align="center">
  <!-- Core -->
  <img src="https://img.shields.io/badge/Python-3.6+-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python" />
  <img src="https://img.shields.io/badge/Next.js-14-000000?style=flat-square&logo=next.js&logoColor=white" alt="Next.js" />
  <img src="https://img.shields.io/badge/TypeScript-5-3178C6?style=flat-square&logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react&logoColor=black" alt="React" />
  <!-- Backend & API -->
  <img src="https://img.shields.io/badge/GraphQL-E10098?style=flat-square&logo=graphql&logoColor=white" alt="GraphQL" />
  <img src="https://img.shields.io/badge/Flask-2-000000?style=flat-square&logo=flask&logoColor=white" alt="Flask" />
  <!-- Security & Auth -->
  <img src="https://img.shields.io/badge/Passkeys-WebAuthn-4F46E5?style=flat-square" alt="Passkeys" />
  <!-- Deploy -->
  <img src="https://img.shields.io/badge/Vercel-Deploy-000000?style=flat-square&logo=vercel&logoColor=white" alt="Vercel" />
  <img src="https://img.shields.io/badge/Render-Backend-46E3B7?style=flat-square&logo=render&logoColor=black" alt="Render" />
  <!-- Meta -->
  <img src="https://img.shields.io/badge/License-Apache%202.0-green?style=flat-square" alt="License" />
  <img src="https://img.shields.io/pypi/v/pytryfi?style=flat-square&color=blue" alt="PyPI" />
</p>

---

## ✨ What is this?

**pytryfi** is a Python interface to the TryFi (Fi) dog collar ecosystem. Use it to:

| Python library | Next.js **PawTracker** app |
|----------------|----------------------------|
| Pets, bases, user details | 📊 7‑day step charts |
| Live GPS & activity | 🗺️ Real-time location map |
| Sleep / nap stats | 🐕 Pet profile & photos |
| LED color, on/off | 🎨 Collar color picker |
| Lost Dog mode | 🔐 PIN + **passkey** 2FA |
| GraphQL queries & mutations | 📍 Charging bases page |

> ⚠️ **Note:** TryFi’s API is undocumented. Endpoints and fields may change without notice.

---

## 🚀 PawTracker — One-Click Deploy

[![Deploy to Vercel](https://vercel.com/button)](https://vercel.com/new/clone)

**Stack:** Next.js (Vercel) → Flask API (Render) → TryFi GraphQL.

- 📊 Interactive step charts · 🗺️ Leaflet maps · 📍 Place & address
- 🎨 Framer Motion · 📱 PWA-friendly · 🔄 Auto-refresh
- 🔐 **Passkey 2FA** (Face ID / Touch ID / security keys)
- 📍 **Bases** page for charging base status

See [Deploy to Vercel](DEPLOY_VERCEL.md), [Deploy Render Backend](DEPLOY_RENDER_QUICK.md), and [website/README.md](website/README.md).

---

## ⚡ Quick start

### 1. Install the Python package

```bash
python -m pip install "pytryfi"
```

### 2. Run PawTracker locally

```bash
export TRYFI_USERNAME=your_email@example.com
export TRYFI_PASSWORD=your_password

pip install -r requirements.txt
cd website && npm install && cd ..
./start.sh
# → http://localhost:3000
```

---

## 📖 Python usage

TryFi uses **GraphQL**. Log in with email + password; the library handles API tokens and sessions.

### Basics

```python
from pytryfi import PyTryFi

tryfi = PyTryFi(username, password)

# Refresh data
tryfi.updatePets()
tryfi.updateBases()
tryfi.update()  # pets + bases

# Pet
p = tryfi.pets[0]
p.updatePetLocation(tryfi.session)
p.updateStats(tryfi.session)
p.updateRestStats(tryfi.session)
p.updateDeviceDetails(tryfi.session)
```

> **Login script / debug:** set `fi_user` and `fi_pass`, then `python login.py` (or F5 in VS Code with breakpoints).

### Collar control

```python
p.setLedColorCode(tryfi.session, 2)
p.turnOnOffLed(tryfi.session, True)
p.setLostDogMode(tryfi.session, True)
```

### Info you can read

| Category | Examples |
|----------|----------|
| **Identity** | `petId`, `name`, `breed`, `gender`, `weight`, `photoLink`, `homeCityState`, DOB |
| **Location** | `currLatitude`, `currLongitude`, `currPlaceName`, `currPlaceAddress`, `areaName` |
| **Activity** | `dailySteps`, `dailyGoal`, `dailyTotalDistance`, `weekly*`, `monthly*` |
| **Rest** | `dailySleep`, `weeklySleep`, `monthlySleep`, `dailyNap`, `weeklyNap`, `monthlyNap` |
| **Device** | `device`, `activityType`, `connectedTo`, `isLost` |

---

## 📈 Historical activity & rest (GraphQL)

```python
from pytryfi import PyTryFi
from pytryfi.common import query
from pytryfi.const import FRAGMENT_ACTIVITY_SUMMARY_DETAILS, FRAGMENT_REST_SUMMARY_DETAILS

tryfi = PyTryFi(username, password)
pet = tryfi.pets[0]
limit_days = 30

# Activity (steps) history
q = (
    'query { pet (id: "' + pet.petId + '") {'
    '  activitySummaryFeed(cursor: null, period: DAILY, limit: ' + str(limit_days) + ') {'
    '    __typename activitySummaries { __typename ...ActivitySummaryDetails start }'
    '  } } }\n' + FRAGMENT_ACTIVITY_SUMMARY_DETAILS
)
r = query.query(tryfi.session, q)
for s in r["data"]["pet"]["activitySummaryFeed"]["activitySummaries"]:
    print(s["start"], s["totalSteps"], s["stepGoal"])

# Rest (sleep/nap) history
q = (
    'query { pet (id: "' + pet.petId + '") {'
    '  restSummaryFeed(cursor: null, period: DAILY, limit: ' + str(limit_days) + ') {'
    '    __typename restSummaries { __typename ...RestSummaryDetails }'
    '  } } }\n' + FRAGMENT_REST_SUMMARY_DETAILS
)
r = query.query(tryfi.session, q)
for s in r["data"]["pet"]["restSummaryFeed"]["restSummaries"]:
    data = (s.get("data") or {}).get("sleepAmounts") or []
    sleep = next((x.get("duration") for x in data if x.get("type") == "SLEEP"), 0)
    nap   = next((x.get("duration") for x in data if x.get("type") == "NAP"), 0)
    print(s["start"], sleep, nap)
```

Use `period: WEEKLY` / `MONTHLY` or up to ~90 days as needed.

---

## 🎙️ Alexa skill (Fi collar LED)

*"Alexa, turn on Ace's collar light"*

- **[Step-by-step](alexa/STEP_BY_STEP.md)** — Full setup from zero
- [Quick path](alexa/SIMPLE.md) — `aws configure` + `./alexa/deploy.sh` + a few console steps

---

## 📁 Docs & links

| Doc | Description |
|-----|-------------|
| [website/README.md](website/README.md) | PawTracker app (Next.js, API, env, structure) |
| [DEPLOY_VERCEL.md](DEPLOY_VERCEL.md) | Frontend (Vercel) |
| [DEPLOY_RENDER_QUICK.md](DEPLOY_RENDER_QUICK.md) | Backend (Render) |
| [TryFi](https://tryfi.com/) | Fi collar & app |

---

## 📜 Version history

| Version | Changes |
|---------|---------|
| **0.0.21** | Better error handling when pet info is missing |
| **0.0.20** | HTTP headers on all GraphQL requests |
| **0.0.19** | Battery health removed (deprecated on new collars); ignore pets with no collar |
| **0.0.18** | Remove deprecated walkversion |
| **0.0.17** | Activity type, current place name, current place address |
| **0.0.16** | Support multiple households (pets & bases) |
| **0.0.15** | Sleep & nap stats (default 0 if unsupported) |
| **0.0.14** | V1/V2 collar compatibility (isCharging) |
| **0.0.13** | Less noisy logging; fix unbound variables in login |
| **0.0.12** | Sentry: capture exceptions only |
| **0.0.11** | Sentry error capture |
| **0.0.10** | `areaName`; fix lat/lng on walk |
| **0.0.9** | LED status from `ledOffAt` vs current time |
| **0.0.8** | Handle unknown location |
| **0.0.7** | Object update bug fixes |
| **0.0.6** | Lost Dog action + `isLost` |
| **0.0.5** | `tryfi.update()`; better error handling |
| **0.0.4** | Update helpers; last-updated times; LED on/off as boolean |
| **0.0.3** | Location on pet; stats; LED color; LED on/off |
| **0.0.2** | Initial TryFi interface (pets, collars, bases) |

---

<p align="center">
  <strong>pytryfi</strong> — Python Interface for TryFi · <a href="https://tryfi.com/">TryFi</a> · Apache 2.0
</p>
