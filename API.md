# TNA-OS-Bitmain — HTTP API Reference

**Firmware:** v0.10.19 · **Updated:** 2026-07-28 · **Applies to:** Antminer S19 / S21 series (all A113D Amlogic control boards)

REST/JSON API for **every Bitmain board running TNA-OS**. This is the integration contract
for building monitoring dashboards, control panels, fleet managers, and automation against a
miner.

> **Companion to the [TNA-OS user manual](README.md).** The manual covers the dashboard and
> every setting for humans; this reference is the machine-facing contract — every endpoint,
> field, and unit. Share the two together.

**One API, every model.** The endpoints, request/response shapes, auth, and error handling
are **identical** across the supported miners — a client written against one works against the
others unchanged. They all run the same firmware from the same source tree; the model is
detected at runtime.

| Model | ASIC | Chips/board | Cores/chip | Boards | Status |
|---|---|---|---|---|---|
| Antminer S19 XP | BM1366 | 110 | 894 | 3 | Validated |
| Antminer S19K Pro | BM1366 | 77 | 894 | 3 | Validated |
| Antminer S19j Pro 104T | BM1362 | 126 | 672 | 3 | Validated |
| Antminer S21 | BM1368 | 108 | 1276 | 3 | Validated |
| Antminer S19 XP Hydro | BM1366 | 204 | 894 | 3 | 🧪 Pending live validation |
| Antminer T21 | BM1368 | 108 | 1276 | 3 | 🧪 Pending live validation |
| Antminer S21 Pro | BM1370 | 65 | 2040 | 3 | 🧪 Pending live validation |
| Antminer S21 XP | BM1370 | 91 | 2040 | 3 | 🧪 Pending live validation |
| Antminer S21+ Hydro | BM1370 | 95 | 2040 | 3 | 🧪 Pending live validation |

**🧪 = the chip family is implemented and auto-detected, but not yet confirmed hashing on that exact
model's hardware.** Other S19j Pro variants (96T, 100T) are auto-detected from the hashboard EEPROM
and fall back to live chip enumeration. The API surface is identical regardless of status.

Where hardware genuinely differs, the **fields** differ — not the endpoints. Fields that don't
apply to the connected hardware read `0` / `-1` / `false` / `""`. **Detect the model from
`deviceModel`** (or its alias `minerModel`); do not infer it from chip count. On a mixed rig
both read `"TNA-OS Hybrid"` — use `boards[].asicModel` for per-board truth.

**Mixed-chip rigs are supported.** A single miner may carry boards of different families (e.g.
two BM1366 + one BM1362). Per-board fields in `boards[]` are authoritative; the top-level
aggregates sum across whatever is fitted.

- **Base URL:** `http://<miner-ip>` (default HTTP port **80**)
- **Content type:** `application/json` (request bodies and all responses)
- **Running version:** `GET /api/system/info` → `version`
- **Web UI:** the same server serves the Angular dashboard at `/`

> **Conventions:** `<miner-ip>` is your miner's LAN address (e.g. `192.168.1.100`). Fields most
> apps need are documented prominently; constant compatibility fields are grouped and labelled
> so the reference stays clean.

---

## 0. Security model

**The HTTP API is intentionally unauthenticated** — no token, no login — so any app on the LAN
can monitor and control the miner. Treat it as fully open to everyone on the network: anyone
who can reach the miner can read telemetry **and** change settings (pools, frequency, voltage),
cut the PSU rail, and restart/reboot it.

Two endpoints are genuinely destructive and worth gating in your own tooling:
`POST /api/system/uninstall` (wipes TNA-OS back to stock) and `POST /api/power/off` (stops
mining and cuts the rail).

The static file server rejects path traversal (`..` or NUL) with **403**, so the API cannot be
walked out of the web root to read system files.

---

## 1. Getting started

```bash
# Read everything (the primary telemetry endpoint)
curl http://<miner-ip>/api/system/info

# Set frequency + fan speed
curl -X PATCH http://<miner-ip>/api/system \
  -H "Content-Type: application/json" \
  -d '{"frequency": 525, "fanspeed": 70}'
```

```python
import requests
BASE = "http://192.168.1.100"

info = requests.get(f"{BASE}/api/system/info").json()
print(f"{info['hashRate_1m']/1000:.2f} TH/s | {info['temp']}°C | {info['power']:.0f} W")

requests.patch(f"{BASE}/api/system", json={"frequency": 525, "fanspeed": 70})
```

### 1.1 Authentication
**None.** All endpoints are open. The miner is intended to run on a **trusted LAN**. There is
no HTTPS and no token — do not expose it to the internet; put it behind your own
firewall/reverse-proxy if remote access is needed.

### 1.2 CORS
Fully open — every response includes:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET,POST,PATCH,OPTIONS
Access-Control-Allow-Headers: *
```
Browser apps can call the API directly. `OPTIONS` preflight returns `204 No Content`.

### 1.3 Status codes & errors
- **Success:** the requested data, or `{"ok": true}` for writes.
- **Failure:** `{"ok": false, "err": "<reason>"}`.
- **Unknown `/api/*` path:** `404` with `{"ok": false, "error": "unknown endpoint", "path": "…"}`.
- **Static files:** `404` for a missing file, `403` for a path-traversal attempt.

Treat a write as **fire-and-forget** and re-read `GET /api/system/info` to confirm the effect.

### 1.4 Polling rate
Telemetry refreshes roughly **once per second**. Polling faster returns the same snapshot.
**1–5 s** is the sensible range for dashboards.

### 1.5 Unit conventions

| Quantity | On the wire | Convert for display |
|---|---|---|
| Hashrate | **GH/s** (number) | `/ 1000` → TH/s |
| Voltage (rail, top-level) | **millivolts** (integer) | `/ 1000` → V |
| Voltage (per-board) | **Volts** (number) | — |
| Temperature | **°C** (number) | — |
| Frequency | **MHz** (number) | — |
| Power | **Watts** (number) | — |
| Fan speed | **percent** (0–100) | — |
| Network rate | **kbps** (number) | — |
| Network total | **MB** (number, since daemon start) | — |
| Difficulty | share difficulty (number) | — |

---

## Quick reference

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/system/info` | Full live state — everything the UI shows |
| GET | `/api/system/asic` | Static per-chip-family metadata (ranges, options, presets) |
| GET | `/api/system/logs` | Last 100 lines of the daemon log |
| GET | `/api/history/len` | How many history samples are buffered |
| GET | `/api/history/data` | Time-series: hashrate, temp, fan, timestamps |
| GET | `/api/system/ethernet/status` | Live Ethernet status (mode, IP, gateway, DNS) — read-only |
| PATCH | `/api/system` | Update settings (fan, freq, voltage, pool, modes, …) |
| POST | `/api/system/restart` | Restart the miner daemon |
| POST | `/api/system/reboot` | Reboot the device |
| POST | `/api/system/power-cycle` | Hard PSU power-cycle |
| POST | `/api/system/uninstall` | Restore stock Bitmain (destructive) |
| POST | `/api/power/on` | Power ON — energise rail + start mining (Idle→Running) |
| POST | `/api/power/off` | Power OFF — stop mining + cut rail (Running→Idle) |
| GET | `/api/power/state` | Power state, auto-power-on, psuKind, last error |
| POST | `/api/board/:id/reset` | Pulse-reset one hashboard |
| POST | `/api/board/:id/enable` | `{"enabled": bool}` enable/disable a board |
| POST | `/api/board/:id/frequency` | `{"override": bool, "mhz": f32}` per-board freq override |
| POST | `/api/psu/hp/fan` | `{"value": 0-65535}` HP PSU fan target (HP-PIC only) |
| POST | `/api/psu/hp/reset-peaks` | Reset HP PSU peak/energy accumulators (HP-PIC only) |
| POST | `/api/psu/calibrate-adc` | Measure the APW12 ADC scale (idle only) |
| POST | `/api/system/ping` | Ping a host, return reachability + RTT |
| POST | `/api/presets` | Save a custom V/F preset |
| DELETE | `/api/presets/:name` | Remove a saved preset |
| PATCH | `/api/thermal/thresholds` | Update warn/hot/danger temps per mode |
| GET | `/api/army/status` | Army (General/Soldier) mode status |
| GET | `/api/army/soldiers` | Connected soldiers (currently `[]`) |
| GET | `/api/stratum_proxy/status`, `/alert/info`, `/influx/info`, `/otp/status` | Feature stubs (see §6) |

---

## 2. Monitoring endpoints (read)

### 2.1 `GET /api/system/info`

The headline endpoint — one JSON object with the miner's live state. The UI polls it ~1/s.

#### Hashrate fields

| Field | Units | Window | Notes |
|---|---|---|---|
| `hashRate` | GH/s | lifetime average | Ramps up over uptime; slow to converge after a reboot. |
| `hashRate_1m` | GH/s | 1-minute rolling | **The UI top-bar reads this.** Smooth and reactive. |
| `hashRate_5m` | GH/s | 5-minute rolling | Longer-window consumers. |
| `hashRate_10m` | GH/s | 10-minute rolling | Feeds the 10m gauge. |
| `hashRate_1h` | GH/s | 1-hour rolling | |
| `hashRate_1d` | GH/s | 24-hour rolling | 0 until the buffer fills. |

All hashrate values use **chip-work per nonce** as the work unit — constant within a chip
family, so every nonce contributes the same value and the number is smooth moment-to-moment:

| Chip family | `work_per_nonce` |
|---|---|
| BM1362 (S19j Pro) | 192 |
| BM1366 (S19 XP) | 256 |
| BM1368 (S21) | 256 |
| BM1370 (S21 Pro/XP) | 256 |

Per-board `boards[N].hashrate` uses the same math scoped to that board's chip family.
The UI computes "Expected" from `frequency × smallCoreCount / 1000` GH/s.

#### Response fields

```jsonc
{
  // ── Mining ──────────────────────────────────────────────
  "hashRate": 13594.0,             // GH/s lifetime
  "hashRate_1m": 17015.0,          // GH/s (top-bar)
  "hashRate_5m": 16030.7,
  "hashRate_10m": 16030.7,
  "hashRate_1h": 0.0,              // 0 until the buffer fills
  "hashRate_1d": 0.0,
  "sharesAccepted": 98,
  "sharesRejected": 0,
  "sharesSubmitted": 97,
  "bestDiff": 226485.5,            // best share difficulty this session
  "bestSessionDiff": 226485.5,     // alias of bestDiff
  "stratumDifficulty": 6053.0,     // current pool-set difficulty
  "poolDifficulty": 6053.0,        // alias of stratumDifficulty
  "hwErrors": 0,                   // cumulative hardware-error count
  "hashEfficiency": 28.25,         // GH/s per watt
  "wPerTh": 35.4,                  // watts per terahash (lower is better); 0 when
                                   //   not mining. Inverse of hashEfficiency.
  "shitcoinDetected": false,       // an alive pool is feeding sub-100-TH work
  "shitcoinPoolIdx": 0,            // which pool index triggered it

  // ── Power ───────────────────────────────────────────────
  "power": 852.5,                  // watts
  "powerMeasured": false,          // true  = read from the PSU's wattmeter (APW17/HP)
                                   // false = MODELLED (profile × (V/profile_V)²).
                                   //   The APW12 has no wattmeter — check this flag
                                   //   before using `power` for efficiency work.
  "minPower": 0.0,
  "maxPower": 5000.0,

  // ── Frequency ───────────────────────────────────────────
  // LIVE vs TARGET: `frequency` is MEASURED (lowest active board) and reads 0 until
  // chips enumerate. `targetFrequency` is what the operator SET — bind settings forms
  // to the target, not the live value.
  "frequency": 185.0,              // MHz LIVE; 0 when idle
  "targetFrequency": 185.0,        // MHz TARGET — bind settings to this
  "defaultFrequency": 185,         // = targetFrequency (kept for older clients)

  // ── Voltage ─────────────────────────────────────────────
  // THREE distinct voltages, all in mV — not interchangeable:
  "voltage": 13330,                // DELIVERED — measured at the PSU output; droops
                                   //   under load; 0 when the rail is off.
  "setpointVoltage": 13304,        // COMMANDED — read back from the PSU (what it is
                                   //   holding). Survives the rail being off. A ramp
                                   //   targets this.
  "targetVoltage": 13300,          // ASKED — the operator's request, before clamping
                                   //   and quantisation to the DAC grid.
  "coreVoltage": 13300,            // mV target (alias of targetVoltage in mV)
  "coreVoltageActual": 13330,      // mV live (alias of voltage); 0 when rail off
  "defaultCoreVoltage": 12940,
  "minVoltage": 11880,             // fitted-PSU range: APW12 11880..15120,
  "maxVoltage": 15120,             //   APW17 11000..15400
  "bypassVoltage": 15000,          // mV — configured value used in psuBypass mode
  "vrFrequency": 0,                // reserved (0)

  // ── Temperatures ────────────────────────────────────────
  "temp": 31.9,                    // °C — hottest board sensor
  "vrTemp": 31.5,                  // °C — second board sensor (near the VR)
  "boardtemp1": 31.9,              // °C — aggregate sensor 1
  "boardtemp2": 31.5,              // °C — aggregate sensor 2
  "overheat_temp": 75.0,           // °C — the normal-mode Danger/reset temperature (becomes
                                   //   bdocOverheatTemp when BDOC is on). The UI also uses it to
                                   //   size the temp dial's redline. It mirrors thermalThresholds
                                   //   .normal.danger; PATCHing overheat_temp is a legacy alias
                                   //   that writes that same Danger threshold.

  // ── Fans ────────────────────────────────────────────────
  "fanspeed": 20,                  // current duty %
  "manualFanSpeed": 20,            // operator-set duty % (survives PID)
  "autofanspeed": 1,               // 1 = PID auto-fan, 0 = fixed duty
  "fanRpm": [2456, 2496, 2556, 2536],  // per-fan RPM (4 fans)
  "fanDuty": [20, 20],             // per-PWM-channel duty %
  "pidTargetTemp": 45.0,           // °C — PID setpoint
  "pidP": 8.0, "pidI": 0.3, "pidD": 0.5,
  "tachWarmingUp": false,          // true briefly at spin-up before RPM is valid

  // ── PSU ─────────────────────────────────────────────────
  "psuModel": "APW12",             // "APW12" | "APW17" | "HP-PIC" | "HP-PMBus" | ""
  "psuHwVersion": "0x76",
  "psuFwVersion": "0x19",
  "psuState": 1,                   // raw PSU state byte (vendor-specific)
  "psuEnabled": true,              // rail EN (GPIO 437) asserted
  "psuBypass": false,              // true = dumb external 12 V, no PSU control
  "psuKind": "auto",               // "auto" | "bypass" | "hp_pic" | "hp_pmbus"

  // ── Power-state machine ─────────────────────────────────
  "powerState": "running",         // "idle" | "running"
  "autoPowerOn": true,             // auto-start mining at boot
  "powerError": "",                // last Power-ON/OFF failure ("" = ok)

  // ── HP common-slot PSU telemetry ────────────────────────
  // Populated only when psuKind is hp_pic/hp_pmbus; -1 / 0 / "" means "not available".
  // The HP PIC only answers telemetry once the rail is ENABLED (standby → hpStatus
  // "standby", all values -1/0).
  "hpVin": -1.0, "hpIin": -1.0, "hpPin": -1.0,
  "hpVout": -1.0, "hpIout": -1.0, "hpPout": -1.0,
  "hpTemp1": -1.0, "hpTemp2": -1.0,
  "hpFanRpm": 0, "hpFlags": 0,
  "hpEnergyWs": -1.0,              // cumulative input energy, watt-seconds
  "hpUptimeS": 0,                  // PSU on-time, seconds
  "hpStatus": "",                  // "live" | "standby" | "absent" | "busy" | ""
  "hpModel": "",                   // EEPROM identity label (readable in standby)

  // ── Identity ────────────────────────────────────────────
  "ASICModel": "Hybrid",           // "BM1366" / … / "Hybrid" for mixed rigs
  "asicModel": "Hybrid",           // alias of ASICModel
  "deviceModel": "TNA-OS Hybrid",  // human model / "TNA-OS Hybrid"
  "minerModel": "TNA-OS Hybrid",   // alias of deviceModel
  "asicCount": 346,                // total chips across active boards
  "smallCoreCount": 281352,        // total small cores (Σ chips × cores/chip)
  "version": "0.10.18",            // firmware version

  // ── Pool / Stratum (flat headline fields) ───────────────
  // The authoritative per-pool data is in `stratum.pools[]` below; these flat
  // fields mirror the primary pool for simple clients.
  "stratumURL": "stratum+tcp://pool.example:3333",
  "stratumPort": 0,                // 0 — parse the port from the URL
  "stratumUser": "bc1q…",
  "fallbackStratumURL": "",
  "fallbackStratumPort": 3333,
  "fallbackStratumUser": "",

  // ── Network ─────────────────────────────────────────────
  "hostname": "TNA-Miner",
  "hostip": "192.168.1.100",
  "macAddr": "02:DC:CE:CB:BE:59",
  "networkMode": "ethernet",
  "ethAvailable": 1,               // eth0 present
  "ethLinkUp": 1,                  // physical link (cable + peer) — real, from sysfs
  "ethConnected": 1,               // interface up
  "ethIPv4": "192.168.1.100",
  "ethMac": "02:DC:CE:CB:BE:59",
  "ethRxKbps": 23.3, "ethTxKbps": 82.0,
  "ethRxTotalMB": 14.0, "ethTxTotalMB": 51.1,

  // ── Control board / system ──────────────────────────────
  "uptimeSeconds": 46,             // daemon uptime
  "systemUptimeSecs": 10324,       // since kernel boot
  "cpuTemp": 35.0,
  "cpuFreqMhz": 667,
  "cpuLoad1m": 3.39, "cpuLoad5m": 1.57, "cpuLoad15m": 1.14,
  "ramTotalMb": 228, "ramUsedMb": 36,

  // ── GPIO / LED ──────────────────────────────────────────
  "ledRed": false, "ledGreen": true,
  "buttonIp": false,               // IP-report button pressed

  // ── Modes ───────────────────────────────────────────────
  "bdocMode": false,               // Big-Dumb-Overclock reset mode
  "bdocOverheatTemp": 95.0,        // °C reset threshold when bdocMode on
  "immersionMode": false,
  "ignoreTempSensorFault": false,  // run despite TMP75 faults (S21 XP has no sensors)

  // ── OLED ────────────────────────────────────────────────
  "oledEnabled": true,
  "oledBrightness": 207,           // SSD1306 contrast 0–255

  // ── Aggregate ───────────────────────────────────────────
  "activeBoards": 3,
  "chipFrequencies": [],           // reserved for per-chip freq telemetry (empty today)

  // ── Nested objects: boards[], stratum{}, thermalThresholds{},
  //    immersionConfig{}, solar{}, army{} — documented below ──
}
```

##### `boards[]` — one object per hashboard slot (array of 3)

```jsonc
{
  "id": 0,
  "state": "active",             // "empty" | "initializing" | "active" | "failed"
  "runtimeState": "active",
  "enabled": true,
  "asicModel": "BM1366",         // per-board chip family — authoritative on hybrids
  "minerModel": "Antminer S19 XP",
  "connector": "J11",            // board connector
  "chips": 110,                  // chips detected
  "chipsPerBoard": 110,
  "domainSize": 10, "domains": 11,
  "frequency": 50.0,             // MHz live (this board)
  "hashrate": 5826.7,            // GH/s (this board)
  "voltageActual": 14234,        // mV live at the board input
  "requestedVoltageMv": 13400,   // mV commanded
  "targetFreqMhz": 110.0,        // MHz target
  "freqOverride": false,         // per-board frequency override active
  "freqOverrideMhz": 0.0,
  "temp1": 30.6, "temp2": 31.5,  // °C — TMP1075 sensors
  "thermalZone": "normal",       // "normal" | "warn" | "hot" | "danger"
  "sensorFault": false,
  "faultReason": null,           // string when state=="failed", else null
  "uartPath": "/dev/ttyS3", "uartBaud": 1000000,
  "resetGpio": 454,
  "nonces30s": 129,              // nonces in the last 30 s
  "rxBytes30s": 6259, "regResponses30s": 440, "dropped30s": 0,
  "attempt": 0,                  // enumeration attempt counter
  "chipNonces30s": [4, 5, 3, …], // per-chip nonce counts (length = chips)
  "chipNonceWindowSecs": 30,
  "chipTemps": [0.0, 0.0, …],    // per-chip die temps °C (length = chips; 0 where the
                                 //   chip family has no per-chip sensor, e.g. BM1366)
  "chipTempWindowSecs": 5
}
```

##### `stratum{}` — pool state

```jsonc
"stratum": {
  "poolMode": 0,                 // 0 = fallback (pool 1 + backup), 1 = multipool (quota-weighted)
  "activePoolMode": 0,           // alias
  "usingFallback": false,
  "totalBestDiff": 226485.5,
  "pools": [                     // up to 8 pools; in multipool each gets quota ÷ Σquota of the hashrate,
                                 //   and a Dead pool's share is redistributed to the survivors
    {
      "url": "stratum+tcp://pool.example:3333",
      "host": "stratum+tcp://pool.example:3333",
      "port": 3333,
      "user": "bc1q…",
      "pass": "x",               // masked
      "protocol": "v1",          // "v1" | "v2"
      "connected": true,
      "status": "alive",
      "accepted": 98, "rejected": 0, "submitted": 98,
      "bestDiff": 226485.5,
      "poolDifficulty": 6053.0,
      "poolDiffErr": false,
      "quota": 1, "effectiveQuota": 100,
      "pingRtt": 0.0, "pingLoss": 0.0
    }
  ]
}
```

##### `thermalThresholds{}`, `immersionConfig{}`, `solar{}`, `army{}`

```jsonc
"thermalThresholds": {
  "normal": { "warn": 60.0, "hot": 62.0, "danger": 75.0 },  // shipped tna-os.toml defaults
  "rebootCooldown": 50.0
},
// Ladder behaviour: warn → fans forced to 100 %; hot → the tripping board's target
// frequency is cut −25 MHz (stepping again every ~30 s while hot, floored at 50 MHz);
// danger → RIG-WIDE protection: the PSU rail is cut (non-bypass) or every board is held
// in reset (bypass), a lockout is set, and mining will not restart until Power ON is
// re-issued or the miner reboots. When BDOC is on, warn/hot are disabled and the single
// bdocOverheatTemp is the only trigger.
"immersionConfig": { "pumpChannel": 2, "pumpSpeed": 100, "radiatorChannel": 1, "tempOffset": 15.0 },
"solar":  { "enabled": false, "pvPower": 0.0, "batterySoc": 0.0, "batteryV": 0.0,
            "acLoad": 0.0, "chargerState": 0, "source": "none" },  // stub subsystem (not yet polled)
"army":   { "mode": 0, "enabled": false, "connectedSoldiers": 0, "totalSoldierHashrate": 0.0 }
```

##### Compatibility fields (constant placeholders)

For compatibility with dashboards written against the common miner-UI schema, the response
also carries these **constant** fields. They do not reflect Bitmain hardware and should not be
relied on: `ssid`, `wifiPass`, `wifiStatus`, `wifiRSSI`, `flipscreen`, `invertscreen`,
`autoscreenoff`, `invertfanpolarity`, `defaultTheme`, `otp`, `shutdown`, `jobInterval`,
`lastResetReason`, `lastpingrtt`, `recentpingloss`, `current`, `proxyDifficulty`,
`stratumEnonceSubscribe`, `fallbackStratumEnonceSubscribe`, `stratum_keep`.

---

### 2.2 `GET /api/system/asic`

Static metadata for the detected chip family (uses the first board's chip type). The UI uses
it for chip-grid dimensions, the profile picker, and default ranges.

```jsonc
{
  "ASICModel": "Hybrid",
  "deviceModel": "TNA-OS Hybrid",
  "asicCount": 346,
  "chipsPerBoard": 110,           // first board's value
  "domainSize": 10, "domains": 11,
  "psuModel": "APW12", "psuHwVersion": "0x76", "psuFwVersion": "0x19",
  "defaultFrequency": 400,        // MODEL default (not the operator target)
  "defaultVoltage": 12000,        // mV, model default
  "absMaxFrequency": 600,
  "absMaxVoltage": 15000,         // mV, chip-family ceiling
  "frequencyOptions": [110, 135, 160, …],         // MHz rungs from the profile table
  "voltageOptions": [11880, 11900, 12000, …, 15120],  // mV — 0.1 V ladder across the
                                                       //   fitted PSU's real range
  "userPresets": [ { "name": "…", "frequency": 500, "voltage": 12.8 } ],
  "swarmColor": "orange"          // compatibility constant
}
```

`voltageOptions` is a clean 0.1 V ladder across the fitted PSU's real range (APW12 34 rungs,
APW17 45 rungs); both PSUs resolve finer than the step so every rung is reachable.

---

### 2.3 `GET /api/history/len` and `GET /api/history/data`

Rolling time-series for the charts. `len` returns the sample count:

```json
{ "len": 11 }
```

`data` returns parallel arrays, all indexed **1:1** by sample (the `_10m` series share one
cadence; `temp_10m`/`fan_10m` pair with `hashrate_10m`):

```jsonc
{
  "hashrate_1m":  [ … ],   // GH/s, recent tail
  "hashrate_10m": [ … ],   // GH/s
  "hashrate_1h":  [ … ],
  "hashrate_1d":  [ … ],
  "temp_10m":     [ … ],   // °C — hottest-board temp, paired with hashrate_10m
  "fan_10m":      [ … ],   // fan duty %, paired with hashrate_10m
  "timestamps":    [ … ],  // ms offsets from timestampBase
  "timestamps_1m": [ … ],
  "timestamps_1h": [ … ],
  "timestamps_1d": [ … ],
  "timestampBase": 1420080686542   // epoch ms; add to each timestamp for absolute time
}
```

---

### 2.4 `GET /api/system/logs`

```json
{ "logs": "…last 100 lines of /var/volatile/tna.log, newline-joined…" }
```

---

### 2.5 `GET /api/system/ethernet/status`

**Read-only.** Live Ethernet state, read from the system each call. The miner runs DHCP
(udhcpc); static-IP configuration is not yet writable.

```jsonc
{
  "networkMode": "ethernet",
  "ethAvailable": 1, "ethLinkUp": 1, "ethConnected": 1,   // real, from /sys/class/net/eth0
  "ethIPv4": "192.168.1.100",
  "ethMac": "02:DC:CE:CB:BE:59",
  "ethUseDHCP": 1,               // 1 = DHCP (always, currently)
  "ethStaticIP": "192.168.1.100",// current IP (informational)
  "ethGateway": "192.168.1.1",   // from /proc/net/route
  "ethSubnet": "",
  "ethDNS": "192.168.1.1"        // first nameserver from /etc/resolv.conf
}
```

---

## 3. Control endpoints (write)

All writes return `{"ok": true}` on success, or `{"ok": false, "err": "<reason>"}`. Re-read
`/api/system/info` to confirm the effect.

### 3.1 `PATCH /api/system`

Update one or more settings. Send only the fields you want to change.

| Field | Type | Meaning |
|---|---|---|
| `frequency` | number (MHz) | Target frequency — ramps to it. |
| `coreVoltage` | number (mV) | Target core voltage — ramps to it. |
| `fanspeed` | number (%) | Manual fan duty (with `autofanspeed:0`). |
| `autofanspeed` | 0/1 | Enable PID auto-fan. |
| `pidTargetTemp`, `pidP`, `pidI`, `pidD` | number | Fan PID tuning. |
| `bypassVoltage` | number (mV) | Voltage shown in `psuBypass` mode. |
| `psuBypass` | bool | Dumb-12 V mode (no PSU control). |
| `psuKind` | string | `auto`/`bypass`/`hp_pic`/`hp_pmbus`. |
| `autoPowerOn` | bool | Auto-start mining at boot. |
| `bdocMode`, `bdocOverheatTemp` | bool / °C | BDOC reset mode. |
| `immersionMode`, `immersionConfig.*` | bool / obj | Immersion cooling. |
| `ignoreTempSensorFault` | bool | Run despite TMP75 faults. |
| `pools` | array | Pool list (`url`, `user`, `pass`, …). |
| `stratumURL`, `stratumPort`, `stratumUser` | string/number | Primary pool shorthand. |
| `hostname` | string | Device hostname. |
| `oledEnabled`, `oledMessage`, `oledBrightness` | bool/string/number | OLED. |

### 3.2 Per-board endpoints

- **`POST /api/board/:id/reset`** — 200 ms reset pulse to one board; other boards keep mining.
- **`POST /api/board/:id/enable`** — `{"enabled": bool}`; disable holds the board in reset.
- **`POST /api/board/:id/frequency`** — `{"override": bool, "mhz": f32}`; per-board override.

### 3.3–3.6 Daemon / device / rail

- **`POST /api/system/restart`** — restart the mining daemon (Linux + fans keep running).
- **`POST /api/system/reboot`** — full kernel reboot.
- **`POST /api/system/power-cycle`** — hard PSU cycle (drops the rail ~5 s). No-op with `psuBypass`.
- **`POST /api/system/uninstall`** — **destructive.** Restore stock Bitmain (wipe TNA-OS, reboot).

### 3.7 Power-state machine

- **`POST /api/power/on`** — energise the PSU rail (EN/GPIO 437) and start mining (Idle→Running).
- **`POST /api/power/off`** — stop mining and cut the rail (Running→Idle). The daemon stays up
  serving telemetry.
- **`GET /api/power/state`** →
  ```json
  { "powerState": "running", "autoPowerOn": true, "psuKind": "auto",
    "powerError": "", "psuEnabled": true }
  ```

### 3.8 HP PSU controls (HP-PIC only)

- **`POST /api/psu/hp/fan`** — `{"value": 0-65535}` set the HP PSU fan target.
- **`POST /api/psu/hp/reset-peaks`** — reset the HP PSU peak/energy accumulators.

### 3.9 `POST /api/psu/calibrate-adc`

APW12 only. Measures the unit's real ADC scale against the DAC law and writes the fit to the
miner log. **Moves the rail — refused unless idle** (`{"ok": false, "err": "power off first …"}`).
Reports; does not auto-apply.

### 3.10 `POST /api/system/ping`

`{"target": "host-or-url"}` → `{"success": bool, "rtt": <ms>}`. Strips stratum URL prefixes.

### 3.11 `POST /api/presets` and `DELETE /api/presets/:name`

Save / remove a named V/F preset. Save body: `{"name", "chip", "frequency", "voltage"}`.

### 3.12 `PATCH /api/thermal/thresholds`

Update the per-mode thresholds and reboot cooldown. All keys optional; send only what changes:

```jsonc
{
  "normal":    { "warn": 60, "hot": 62, "danger": 75 },  // °C — the live ladder (see §2.1)
  "immersion": { "offset": 15 },                          // °C added to all three when immersion on
                                                          //   (may also set explicit warn/hot/danger)
  "bdoc":      { "bdocOverheat": 95 },                    // °C — sole trigger while BDOC active (50–120)
  "rebootCooldown": 50                                    // °C — cool-to target before a full reboot (20–80)
}
```

Applies immediately and persists to `/config/tna-os.toml`. Recall that `danger` is a **rig-wide**
rail cut + lockout, not a single-board reset (see §2.1).

---

## 4. Army / Soldier mode

- **`GET /api/army/status`** →
  ```json
  { "mode": 0, "enabled": false, "port": 2121,
    "connectedSoldiers": 0, "totalSoldierHashrate": 0.0 }
  ```
- **`GET /api/army/soldiers`** — array of connected soldiers (currently `[]`).

---

## 5. Feature stubs

These answer with a fixed shape and exist so clients don't 404 while the features are pending:

| Endpoint | Response |
|---|---|
| `GET /api/stratum_proxy/status` | `{"running": false}` |
| `GET /api/alert/info` | `{"enabled": false}` |
| `GET /api/influx/info` | `{"enabled": false}` |
| `GET /api/otp/status` | `{"enabled": false}` |

---

## 6. Integration recipes

**Poll + control (Python):**
```python
import requests
BASE = "http://192.168.1.100"

def stats():
    i = requests.get(f"{BASE}/api/system/info").json()
    return {
        "ths":      round(i["hashRate_1m"] / 1000, 2),
        "temp":     i["temp"],
        "watts":    round(i["power"]),
        "measured": i["powerMeasured"],   # False = watts are modelled, not metered
        "wPerTh":   round(i["wPerTh"], 1),
        "boards":   [(b["asicModel"], b["chips"], b["thermalZone"]) for b in i["boards"]],
    }

requests.patch(f"{BASE}/api/system", json={"frequency": 500, "coreVoltage": 13000})
```

**curl cookbook:**
```bash
# Health check
curl -s http://<ip>/api/system/info | jq '{ths:(.hashRate_1m/1000), temp, watts:.power, state:.powerState}'
# Per-board summary
curl -s http://<ip>/api/system/info | jq '.boards[] | {id, asicModel, chips, thermalZone, temp1, temp2}'
# Cut the rail (stops mining; daemon stays up)
curl -s -X POST http://<ip>/api/power/off
# Bring it back
curl -s -X POST http://<ip>/api/power/on
```

---

## 7. Gotchas

- **Bind settings forms to the TARGET fields.** `frequency`/`coreVoltageActual` are *measured*
  and read `0` when idle; `targetFrequency`/`coreVoltage` are what the operator set.
- **Three voltages, all mV, not interchangeable:** `voltage` (delivered, droops under load),
  `setpointVoltage` (commanded, read back), `targetVoltage` (asked, pre-quantisation).
- **`power` may be modelled.** Check `powerMeasured` — on the APW12 there is no wattmeter, so
  `power` (and therefore `wPerTh`) is derived from the profile, not measured.
- **Hybrid rigs:** top-level identity reads `"Hybrid"`; use `boards[].asicModel` for per-board
  truth. Chip counts and cores differ per board and the aggregates sum across them.
- **Per-board `voltage` is Volts; top-level voltages are millivolts.**
- **Per-chip temps:** `boards[].chipTemps` is `0` for chip families without a per-chip sensor
  (e.g. BM1366); board temps `temp1`/`temp2` are always real.
- **Unknown `/api/*` paths return `404`**, not `200` — a real endpoint check.
