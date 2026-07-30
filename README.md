# TNA-OS-Bitmain

### The firmware your Antminer was supposed to ship with.

**Version 0.11.0** · Antminer S19 series · Antminer S21 series compatible with all A113D Amlogic control baords.

---

Stock firmware treats your miner like an appliance. TNA-OS treats it like a **machine you
own.**

It rips out the stock software down to the metal and rebuilds the whole thing — our own Linux
kernel, our own root filesystem built from source, the mining engine, the dashboard, the
network stack, the safety system — into one fast, modern, self-hosted operating system. No
cloud account. No phone-home. No leash. No Dev fees. It boots straight into a slick web control centre
served **by the miner itself**, and it answers to nobody but you.

One firmware. Every board. Total control.

> **⚡ Flash it now → [TNA Flasher](https://lnbits.molonlabe.holdings/tnaflasher/public)**
> · 🔌 [API reference](API.md)

> **🧪 Honest heads-up:** TNA-OS is under active, ongoing development. The core mining,
> monitoring, pool, power and safety features below are solid and in daily use — but anything
> tagged **🧪 experimental** is still being hardened and may change or misbehave build to build.
> The hype is real; where something's still cooking, it says so.

---

## This is what control feels like

**Whole racks of hash chips. All the power in your hands.**
Three hashboards of BM1362 / BM1366 / BM1368 / BM1370 silicon — hundreds of SHA-256 chips per
machine. TNA-OS gives you full control of the chain: the global frequency and voltage every
board ramps to, factory-tested presets you can save and recall, and **per-board frequency
overrides** to run one board at its own clock while the rest hold steady. Balance a rig. Nurse
a weak board. Push a strong one. This is control stock firmware won't even show you.

**One brain runs mixed silicon.**
TNA-OS runs a **hybrid rig** — different ASIC families on the same
miner (say two BM1366 boards and one BM1362) — as one machine. It reads each board's identity
from its EEPROM, enumerates it, and tunes, measures and reports it on its own terms; the
dashboard sums whatever is fitted. Detected and handled automatically.

**Sub-second telemetry. Watch every board breathe.**
Live hashrate smoothed across 1-minute, 10-minute, hourly and daily windows. Per-board
temperatures, and per-chip die temps on the families that carry a sensor. Real fan tachometers,
edge-counted. Power draw and efficiency in W/TH. Share counts per pool. All refreshed **every
second** and drawn straight onto built-in history charts. No external logger. No spreadsheet.
Just open a browser and *see everything.*

**Real voltage control, straight over I²C.**
TNA-OS talks to your PSU directly — **APW12, APW17, and HP common-slot server PSUs** — auto-
detects which one is fitted, and ramps the core voltage to your setpoint in smooth 50 mV steps.
The APW17 reports its own wattage from an on-board meter; the HP PICs stream live V / A / W /
temperature / fan RPM. Pick your PSU in one dropdown and go. Or just set to bypass for an agnostic power input with no control or monitoring expectations.

**Failover and multipool, built in.**
Run **up to 8 pools**. Fallback that catches a dead pool without dropping a beat, or quota-
weighted multipool that splits your hashpower exactly how you want and instantly redistributes
it when a pool falls over. Stratum V1 and V2 (encrypted + authenticated), per pool. Enter a pool
by hostname or IP — the resolver handles the rest.

**Kill the rail. Bring it back. No reboot.**
Cut power to the hashboards and the *brain stays alive* — dashboard, network, telemetry, all
still up. Bring it back and watch voltage and frequency ramp from a safe floor and reconnect to
your pools with a fresh session, automatically. Instant on/off for the part that burns watts,
zero downtime for the part that runs the show.
alternatively disable individual hashboards. 

**A safety system that never sleeps.**
A layered thermal ladder that forces the fans to full, then steps the hot board's clock down,
then cuts the whole rail and locks out before anything cooks. A PID fan controller holding your
target temperature to the degree. Immersion-cooling mode for the tank builds. It's engineered to
run **unattended, for months, and not care.** - NOTE: the thermal danger psu cut off is not availble in psu bypass mode. 

**Bitcoin only — or it stops.**
Point a pool at non-Bitcoin SHA-256 work and TNA-OS catches it, throttles every board to a safe
100 MHz, and lights up a warning on the dashboard. Checked *per pool* with a debounce, so a decoy
can't sneak past. Your hashpower mines what you told it to. Full stop. for more info read about <a href="https://github.com/CryptoIceMLH/TNA-OS-CANAAN/blob/main/shitcoin-detector-nbits.md" target="_blank">"nbits" here </a>

**Wired or wireless — run your miners anywhere.**
No Ethernet drop where the rig lives? Plug a **USB WiFi dongle** into the control board and TNA-OS
brings the miner online over **WiFi**. Ethernet wins when a cable's in; otherwise the miner raises
its own **`TNA-Setup`** hotspot with a captive setup page — connect, type in your WiFi, done. And it
won't burn a single watt hashing until it actually has a network. Flashing still uses Ethernet.

**Open to everything. Owned by you.**
A clean REST/JSON API on the same address as the dashboard. Home Assistant in three lines of YAML.
And the whole thing runs **100% local** 

---

## Every Antminer, one brain

Plug a board in and TNA-OS reads its EEPROM, identifies the model, configures the ASIC registers
for that chip family, and starts mining. Same firmware, same dashboard, same API on every one:

| Model | ASIC | Chips/board | Cores/chip | Boards | Status |
|---|---|---|---|---|---|
| Antminer S19 XP | BM1366 | 110 | 894 | 3 |
| Antminer S19K Pro | BM1366 | 77 | 894 | 3 |
| Antminer S19j Pro 104T | BM1362 | 126 | 672 | 3 |
| Antminer S21 | BM1368 | 108 | 1276 | 3 |
| Antminer S19 XP Hydro | BM1366 | 204 | 894 | 3 |  🧪 Pending Live Validation on hardware |
| Antminer T21 | BM1368 | 108 | 1276 | 3 | 🧪 Pending Live Validation on hardware
| Antminer S21 Pro | BM1370 | 65 | 2040 | 3 |  Pending Live Validation on hardware
| Antminer S21 XP | BM1370 | 91 | 2040 | 3 | 🧪 Pending Live Validation on hardware
| Antminer S21+ Hydro | BM1370 | 95 | 2040 | 3 | 🧪 Pending Live Validation on hardware

**Mixed-chip (hybrid) rigs are supported** — a single miner may carry boards of different
families, and the per-board view is authoritative; the top-level numbers sum whatever is fitted.

> **One requirement:** TNA-OS runs on the **Amlogic A113D control board** (the AArch64 quad-A53
> board fitted to the models above). Units built on a *different* control board — some Hydro
> variants ship a Xilinx-based board — cannot run TNA-OS regardless of their chip family. Check
> your control board, not just the model name.

---

## Wired or wireless — your call 📶

No Ethernet where your miners live? Plug a **USB WiFi dongle** into the control board and TNA-OS
brings the miner online over **WiFi** — no cable required. Ethernet always wins when a cable is
connected; otherwise the miner raises its own **`TNA-Setup`** hotspot with a captive setup page
(connect from your phone and it pops up on its own), you enter your WiFi credentials, and it reboots,
joins your network, and mines. Supported dongles (Realtek): **RTL8723DU · RTL8821CU · RTL8811CU ·
RTL8822BU · RTL8812BU · RTL8812AU · RTL8811AU**.

A miner with **no network yet won't waste power** — it sits idle (dashboard and hotspot still
reachable) until Ethernet or WiFi is online, then starts mining on its own.

---


# User Manual

Everything below is the practical, do-it guide. Skim the headers, jump to what you need.

## Getting started

### 1 · Flash the firmware


**👉 [TNA Flasher — lnbits.molonlabe.holdings/tnaflasher/public](https://lnbits.molonlabe.holdings/tnaflasher/public)**

Its strongly advisable to install TNA OS over the clean stock firmware found on the flasher. Download the archive, extract it, and copy the 3 files to the root of an SD card ( mandatory FAT32 format so mind what size flash drive you use) then plug in usb drive to micro usb port and boot the miner up. Wait aprox 10sec until the Green LED comes on and stays on. Then power off miner and remove USB flash drive. This will set your control board up ready for flashing TNA-OS. 

1. Make sure the miner is **powered on and on the same network** as your PC (wired ethernet).
2. Download and run the **TNA Flasher** for your OS.
3. Enter the miner's **IP address** — the flasher pings it first, so a bad address never burns your code.
4. Paste your **flash code** and let it run. The flasher lays down the complete TNA-OS image, and reboots.
5. When it comes back up, it's TNA-OS. First boot pulls an IP by DHCP.

> **⚠ Flash over Ethernet — not WiFi.** The flasher connects to your miner over your **wired
> network**, so the miner must be on Ethernet while you flash it. A miner that's only on WiFi (or
> currently showing the `TNA-Setup` setup hotspot) **can't be flashed** until you plug it into
> Ethernet. WiFi is for *running and configuring* the miner — never for flashing it.

> **One image, every model.** 
> the firmware works out which miner hashboard it's controlling by itself.



### 2 · Find your miner

- On first boot the miner pulls an IP address from your router automatically (DHCP).
- Get that address from your router's device list, or read it off the miner's display.
- Open **`http://<miner-ip>/`** in any browser. That's your dashboard.
- TNA-OS **always uses DHCP** (static addressing isn't implemented yet). If you want a fixed
  address, set a **DHCP reservation** on your router against the miner's MAC.

---

## Using the web interface

The UI has three main pages: the **Dashboard** (live monitoring), **Settings** (all
configuration), and **System** (power, reboot, live logs). Army and Solar pages appear for those
features.

Every dashboard card lives on an **editable grid** — hit **✎ Edit layout** (top of the page),
then drag a card by its header and resize it from the bottom-right corner. **↻ Reset** restores the default layout. Your arrangement is saved in that browser. Telemetry refreshes about once a second.

---

## The Dashboard (live monitoring)

Two banners can take over the top of the page:

- **⚠ BDOC MODE ACTIVE** — shown whenever BDOC overclocking is on, stating the current reset
  temperature, so you can never forget that safety limits are relaxed.
- **💩 SHITCOIN DETECTED** — a pool is feeding sub-100T (altcoin) work; every board has been
  throttled to **100 MHz** to protect power. It names the offending pool. Point the miner back at a real BTC pool to clear it.

Each card below is a tile you can move or resize.

### Hash Rate
The headline number is your hashrate in **TH/s** — the **1-minute rolling average**, derived from real chip work (nonces × per-family work-per-nonce) over a 60-second window, so it reacts fast but stays smooth. **Expected** is the theoretical rating at your current frequency (chip cores × MHz): a reference, not a chip reading; a healthy miner usually beats it.

### Miner Status
Live ring gauges, updated every second: **Freq** (MHz), **PSU** voltage (V), **Temp** (hottest board °C), **Fans** duty (labelled **Pump** in immersion mode), **ASICs** (enumerated chip count), and **Power** (W — hidden when you run a bypass PSU, since there's nothing to measure). Above the gauges: your target frequency, set voltage, and live Ethernet throughput (↓↑ kbps and cumulative MB). **BDOC** and **IMMERSION** badges light up here when active.

### Pools
A status badge per pool — green connected, amber disconnected, red dead — each showing its
effective quota %.

### Shares
**Accepted / Sent / Rejected** since the daemon started, plus a per-pool breakdown in multipool.
A high rejection rate usually means a V/F mismatch, network latency, or pool difficulty set too low.

### Best Difficulty
The highest-difficulty share this miner has ever found — **all-time**, and **since boot** (per pool in multipool). If it ever equals the current Bitcoin network difficulty, **you've found a block.**

### Bitcoin Network
Live network stats — block height, difficulty, estimated network hashrate — pulled from a public source (mempool.space), not your pool. Two rows are **clickable toggles**: the first flips between **sats/min** and your **solo-block odds** ("1 in N years"); the second flips between a playful **fiat-printed** counter and your **session BTC mined**. (These estimates assume 144 blocks/day and the 3.125 BTC reward, and the fiat counter is an editorial gag — treat them as fun, not accounting.)

### Hash Rate Chart
Your hashrate over time with six toggleable legend traces: **1m / 10m / 1h / 1d**, plus **Temp** and **Fan** overlays (both off by default, drawn on a second axis). Click a coloured legend dot to toggle each. A trash icon clears the stored history.

### Hardware Schematic
The per-board and per-chip view, the PSU panel with **Power ON / OFF**, and the miner's identity rows. Covered next.

---

## The Hardware Schematic (per-board & per-chip)

Each active hashboard is drawn as a grid of chips, with a header showing `chips × ASIC`, the board frequency, its voltage, and its TH/s.

**Chip display dropdown** (per board) — *"Pick what every chip on this hashboard shows. Click an individual chip to override just that one."* The options:

| Option | What each chip cell shows |
|---|---|
| **Health** | Colour only — **green** = the chip enumerated and is part of the chain, **grey** = a slot that didn't come up. (No number; health-by-hashrate colouring is coded but dormant until per-chip nonce attribution lands.) |
| **Hashrate** | Shows `--` for now — 🧪
| **Temp °C** | The **real per-chip die temperature**, 🧪 where that chip reports one; `--` until the first sample or on families without a per-chip sensor. |
| **Freq MHz** | The **board's** operating frequency, shown on every chip (there's no per-chip write path, so all chips read the same value). |
| **Chip ID** | The chip's index on the board. |

Click any single chip to cycle just that one through the metrics; changing the board dropdownresets all the per-chip overrides on that board.

**Per-board controls** — each *stages* your edit; nothing hits hardware until you press **✓ Apply**on that card (**Cancel** discards):

- **↻ Reset** — a 200 ms reset pulse to that board only; its chips re-init while the other boards   keep mining. Confirms first.
- **Enabled / Disabled** — Disabled holds the board in reset until you re-enable it.
- **Freq override** — run this board at its own frequency instead of following the global setting. Toggle it on and a slider (50–700 MHz) appears.

**Board status** — each board shows a state (**ACTIVE / DISABLED / RE-INIT / OVERHEAT / FAILED / EMPTY**) and, when it leaves normal, a thermal zone badge (**WARN / HOT / DANGER**). A **⚠** marks
a board whose TMP75 sensor stopped responding (firmware falls back to the last good reading —check the I²C wiring). Temperature bars show the coldest and hottest reading on the board.

**PSU panel & Power ON / OFF** — the rail controls live here, not per board:
- **Power ON** energises the PSU rail, confirms output is live, then enumerates the boards and starts mining (~15 s).
- **Power OFF** stops mining, holds the boards in reset and cuts the main rail. It stays available for any non-bypass PSU so you can cut a rail that's still live even while the miner shows idle.
- On an HP PSU the panel also streams live V / A / W / temperature / fan and offers a fan-target  and peak-reset — the HP PIC only reports once the rail is enabled, so it reads **STBY** in standby.

---

## Settings — performance tuning

The **Settings** page holds all configuration. Most controls **stage** together and are pushed by the **Save** button at the top, which persists everything to the miner's config file. A red **"Restart required after save"** appears when a change (hostname, pool mode,  the difficulty values, fan-polarity) needs a daemon restart to take effect.

**Complexity levels.** A small **⚠ triangle** in the Mining card header switches the UI between three levels (near-invisible in Safe mode by design): single-click cycles **Safe ↔ Advanced**, double-click jumps to **Pro**. **Safe** gives dropdowns of factory-tested values; **Advanced** lets you free-type frequency/voltage and reveals the fan PID gains and the Difficulty card; **Pro** unlocks fan-polarity invert and BDOC (below).

- **Frequency (MHz)** — the target clock all boards ramp to. Higher = more hashrate, more heat,  more power. In Safe mode you pick from a drop down; in Advanced you can type any value (red when it exceeds the chip's absolute max).
- **Core Voltage (mV)** — the PSU output delivered to every board, in millivolts (e.g. **13300** =  13.3 V). **It must match the factory profile for your chosen frequency** — too low starves the  chips, too high can damage them. Only change it if you know the V/F table.
- **Custom presets** — save the current frequency + voltage pair under a name and recall it in one   click later. Presets are **scoped per chip family** and persisted on the miner. *Load* only fills  the form — press the top **Save** to apply; **Delete** removes one.
- **PSU** — tell the firmware which supply is fitted, then Save and reboot:
  - **Stock APW (APW12 / APW17) — auto-detect** — Bitmain's PSU; the firmware controls voltage and     on/off over I²C. The APW17 reports real wattage; the APW12's power is modelled.  - **HP common-slot — PIC (HSTNS-PL11 / DPS-1200FB)** — our telemetry PSU; on/off via the EN line,   with live V / A / W / temp / fan. (Output voltage is set by the EN line — fine trim over I²C is a bench-calibration item, so voltage changes are a no-op on HP today.)
  - **HP common-slot — PMBus (DPS-750RB)** — standard PMBus telemetry.
  - **Bypass — plain 12 V PSU (no control, no data)** — you feed your own DC supply; no start/stop, no telemetry. A **PSU Voltage (V)** field (10–16 V) appears so the UI can *estimate* power draw — the firmware never sets voltage in bypass.

### 🔥 BDOC — expert overclocking (Pro level)
**BDOC (Balls-Deep Overclocking)** removes *all* frequency and voltage safety clamps for full manual control. It's reached by taking the UI to Pro, and the firmware makes you own it — the confirmation lineage ends in a dialog literally titled *"FUCK AROUND AND FIND OUT"*, because **wrong values can instantly destroy chips or brick the board.** With BDOC on, the normal Warn/Hot/Danger ladder is disabled and a single **BDOC overheat temp** (40–120 °C, default 95) is the only trigger; the dashboard flies a permanent **BDOC MODE ACTIVE** banner. Experts only, entirely at your own risk.

---

## Settings — pools & work distribution

**Pool Mode** decides how your hashpower is shared:

- **Fallback (Primary + Backup)** — mine on Pool 1; if it dies, auto-switch to Pool 2, and switch   back when Pool 1 recovers.
- **Multipool (up to 8 pools with quotas)** — split hashrate across up to eight pools by quota.
  **Add Pool** creates another slot (2–8). Quotas are relative: a pool gets **its quota ÷ the sum   of all quotas** (2 : 1 : 1 → 50 % / 25 % / 25 %).

Each pool has:

| Field | What to enter |
|---|---|
| **Stratum Host** | Pool hostname **or** IP — no `stratum+tcp://` prefix, no port. |
| **Stratum Port** | The pool's TCP port (commonly 3333, 4444, 25). |
| **Username** | Your worker/user — usually your BTC address, optionally with a `.worker` suffix. |
| **Password** | Most pools accept `x`. (Masked; a show/hide eye reveals it.) |
| **Quota** (multipool) | 1–100. Higher = more hashrate to this pool. |
| **Protocol** | **Auto-detect** (tries V2, falls back to V1), **Stratum V1**, or **Stratum V2**. |
| **Pool Authority Key** | **V2 only** — the pool's authority public key, from its V2 connection page. Without it the miner can't verify the pool. |

Plus **Enable Extranonce Subscribe** (V1/auto — the pool pushes fresh work without a reconnect) and **Stratum TCP Keepalive** (heartbeat for pools that drop idle clients — safe to leave on).

**When a pool goes down.** In **multipool**, the dead pool's quota is **redistributed to the
survivors** automatically after a 60-second grace period — each pool's card shows its *effective* quota next to its *configured* one so you can watch it happen. In **fallback**, the miner moves to the backup and returns to the primary when it recovers. (A worker the pool doesn't recognise shows up as rejects — that's an account/worker-name issue with the pool, not the miner.)

---

## Settings — fans & cooling

**Fan Controller** runs in one of two modes:

- **Manual** — two duty sliders, because the boards have two PWM banks: **Fans 1‑2** (PWM channel 0)   and **Fans 3‑4** (PWM channel 1). Below **40 %** you get a ⚠ *Danger* badge (usually too little airflow for normal mining); at **100 %** a *MAX* badge (no headroom left for the thermal ladder to push harder). In immersion mode these become **Pump** and **Rad Fan**. 
- **PID (Auto)** — set a **Target Temp** (°C, 30–80) and the controller holds the *hottest* board at that temperature by varying fan duty. The **P / I / D** gains (Advanced+) come pre-tuned; only touch them if you see fans hunting (oscillating) or responding slowly — **P** = how hard fans react, **I** = corrects long-term drift, **D** = damps rapid swings.
- **Invert Fan Polarity** (Pro) — flip this if a fan runs backwards (0 % spins fast, 100 % slow); some non-Bitmain fans need it.

---

## Settings — thermal thresholds (the temperature cards)

This is the per-board **thermal ladder** in Normal mode — each a °C value:

| Threshold | What happens at it |
|---|---|
| **Warn** | Fans are forced to **100 %** and a warning is logged. |
| **Hot** | That board's frequency is cut **−25 MHz**, stepping down again every ~30 s while it stays hot (floored at 50 MHz). Other boards are untouched. |
| **Danger / Reset** | **The whole rig protects itself** — TNA-OS cuts the PSU rail (or, on a bypass PSU, holds every board in reset), records the reason, and **locks out** so mining won't restart until you re-issue Power ON or reboot. Deliberately rig-wide, not one board. |

Plus:
- **Immersion offset** — added to all three thresholds when immersion mode is on (and the Hot step
  skips the frequency cut, since oil carries heat far better than air).
- **BDOC reset** (Pro) — the single trigger used while BDOC is active; the Warn/Hot ladder is disabled then, and within 5 °C of it the fans are forced to full.
- **Reboot cooldown** — before a full reboot the miner waits for every board to cool to this
  temperature (hard timeout 120 s).

Live, each board shows its zone (WARN / HOT / DANGER) as it crosses each step. **Save thresholds** applies immediately.

---

## Settings — behaviour, network & display

**Behaviour**
- **Auto Power-On** — **ON** (the shipped default) energises the rail and starts mining at boot;
  **OFF** boots **idle** — UI and telemetry up, hashboards off until you press Power ON. Takes   effect on the next boot. (A deliberate Power OFF also sticks across that boot cycle.)
- **Immersion Mode** — dielectric-fluid cooling: raises every thermal threshold by the offset and remaps the PWM channels so one drives a fixed-duty **pump** and another a **PID radiator fan**.
  Its sub-config (pump channel + duty, radiator channel, temp offset) has its own **Save immersion config** button.
- **Ignore Temp Sensor Fault** — keep hashing through TMP75 I²C errors. Use **only** if the board genuinely has no sensors (e.g. an S21 XP); with a real probe, a fault means a real cooling problem.

**Difficulty (Advanced/Pro)** — **Initial Suggested Stratum Difficulty** (default 1000; the pool has final say), plus **Job Interval** (default 20 ms), **VR-Frequency** (version-rolling rate, with the resulting wrap-around time shown live), and **Proxy Miner Difficulty** (default 10000, used only when TNA-OS acts as a stratum proxy — see Army). Leave these alone unless you know why you're changing them.

**Network** — **Hostname** (shown in the header, in pool worker reports, and on your router's device
list; reboot to apply). The card shows your **live connection at a glance — Ethernet, WiFi (with the
joined SSID + signal + dongle chipset), or the `TNA-Setup` setup hotspot** — plus status, IP and MAC.
Addresses are assigned by **DHCP**; for a fixed one, set a **DHCP reservation** on your router.

**WiFi (optional USB dongle) — how the setup hotspot works, step by step.** With **no Ethernet
cable** connected, plug a supported dongle into the control board and:

1. The miner raises an **open** WiFi hotspot named **`TNA-Setup`** — no password needed to join it.
   (Give it up to ~1 minute after boot to appear.)
2. On your phone or laptop, connect to **`TNA-Setup`**. The setup page usually pops up on its own
   (captive portal); if it doesn't, just open a browser to **`http://192.168.4.1`**.
3. The **WiFi Setup** card asks for **your** home/office network name (SSID) and password
   (min 8 characters). Enter them and press **Connect**.
4. The miner saves the credentials, **reboots, and joins your WiFi.** Put your phone/laptop back on
   your normal network and find the miner at its new DHCP address (check your router, or the
   miner's screen).

While it has no network, the miner stays **idle — no hashing — until Ethernet _or_ WiFi comes
online**, then it starts mining automatically (so it never burns power with no pool to reach and
stays reachable for setup). Remember: **flashing always needs Ethernet** (see *Flash the firmware*)
— the hotspot is for configuring WiFi, not for flashing.

> Pool hostnames resolve reliably: as of v0.10.19 the DHCP client falls back to your **gateway as the
> resolver** when a lease carries no DNS server, so pools entered by hostname keep working even on
> networks that don't hand out DNS.

**Display & lights** — OLED and status-LED behaviour are managed by the firmware and are **not exposed on the settings page** in this build.

---

## 🧪 Per-chip tuning (experimental)

The schematic has a **⚙ Per-chip freq** button the UI itself tags **"WIP — don't use."** It opens a per-chip grid, but **Apply is permanently disabled** ("Not yet implemented in firmware — the daemon accepts a per-board override only"). Per-chip clocking isn't validated on hardware yet. **Use the global frequency plus per-board override for real mining;** treat per-chip tuning as a preview.

---

## The System page — power & maintenance

Three escalating recovery actions — pick the smallest one that fixes the problem — plus power and uninstall:

- **Power ON** — energise the rail, enumerate chips, start mining (~15 s). No confirmation.
- **Power OFF** — stop mining, hold boards in reset, cut the rail. The control board, UI, network   and telemetry stay online — only the "heater" goes off. Confirms first.
- **Restart Daemon** (~30 s) — re-execs the mining daemon only; Linux, fans, network and UI stay up.
  Try this first if mining looks stuck.
- **Power-Cycle PSU** (~5 s off) — a hard PSU off-then-on; the control board stays up, only the chips lose power and re-init. Disabled on a bypass PSU (no EN control). Confirms first.
- **Reboot Control Board** (~60–90 s) — a full kernel reboot; everything cold, fans respool. Last resort. Confirms first.
- **Uninstall TNA-OS** — restore the miner to stock Bitmain firmware. Runs locally on the miner (no PC needed), **double-confirmed** — the second step makes you type `RESTORE`. Destructive and irreversible from here (you'd re-flash to get TNA-OS back).

The overview strip shows the model + ASIC, firmware version, daemon uptime, IP, MAC, last boot reason, the **power state** (Idle vs Running), and the real **PSU rail** (EN) state — which can be ON even while mining is Idle.

**Realtime Logs** — a live stream you can shape: toggle it on, **pause** the auto-scroll, filter by **source** (main, chain, controller, rx, stratum, psu, safety, api, config), flip to **Errors only**, text-filter, and **Capture → Download** a `.txt` for support.

---

## Saving changes

Most Settings controls stage together — click **Save** at the top to push and persist them. A red **"Restart required after save"** means the change needs **Restart Daemon** to take effect. A few controls save on their own dedicated buttons and apply immediately: **Thermal thresholds**, **Immersion config**, and **Presets**.

---

## Automation & integrations

TNA-OS serves a full REST/JSON API right alongside the dashboard:

```bash
# Read everything
curl http://<miner-ip>/api/system/info

# Change frequency + voltage
curl -X PATCH http://<miner-ip>/api/system \
  -H "Content-Type: application/json" \
  -d '{"frequency": 525, "coreVoltage": 13300}'
```

Great for custom dashboards, fleet tools, and **Home Assistant** (a REST sensor is three lines of YAML). One API, every model — a client written against one miner works against the others unchanged. Full reference — every field, every endpoint, ready-to-paste recipes — is in the
**[API reference](API.md)**.

---

## Security — read this

The access model is deliberate and simple:

- The HTTP **API and dashboard are open by design** so any app on your LAN can monitor and control
  the miner. That's powerful, and it means: ✅ **run TNA-OS on a trusted home network**, and ⛔   **never expose the miner straight to the internet** — put it behind your own VPN or firewall.

- Your **pool credentials** live on the miner and are **never** returned by the API or written to the logs.

> **🧪 OTP (2FA) is scaffolded, not yet live.** The UI includes an authenticator-code (TOTP) flow,
> but the firmware currently reports OTP as disabled, so it doesn't gate anything yet. It's wired for > a future release; don't rely on it as protection today.

---

## Troubleshooting

| Symptom | What to do |
|---|---|
| Can't find the miner | Check your router's device list, or read the IP off the miner's screen. |
| Dashboard won't load | Confirm the IP, then hard-refresh the browser (clear site data). |
| Pool by hostname won't connect | Fixed in current builds (gateway-DNS fallback). If it persists, check the pool host, or enter the pool by IP. |
| Low hashrate right after Power ON | Normal — voltage and frequency ramp up from a safe floor over the first minutes. |
| Shares getting rejected | Usually a pool account / worker-name issue — double-check your worker with the pool. |
| A board shows a ⚠ | Its temperature sensor stopped responding; check I²C wiring. Firmware uses the last good reading. |
| Miner throttled to 100 MHz | Shitcoin protection — a pool served non-Bitcoin work. Point it back at a Bitcoin pool. |
| Rig cut power and locked out | The Danger threshold tripped. Let it cool, fix airflow, then press Power ON. |
| Flash fails on an old build | Some miners carrying stale on-board data need a factory reset to stock first, then flash TNA-OS. |
| WiFi hotspot (`TNA-Setup`) doesn't appear | Use a supported Realtek dongle, boot with **no Ethernet cable**, and give it ~1 min. Flashing itself must always be done over Ethernet. |
| Miner shows Idle and won't mine | It has no network IP yet — plug in Ethernet or finish WiFi setup. TNA-OS won't hash without a reachable network ("no IP, no mining"). |

---

## What's next — and what's on the bench 🧪

TNA-OS is under active development. Everything above is **shipping and working today, except the items tagged 🧪**. Still cooking:

- **🧪 Per-chip tuning** — the editor is in the UI but its write path isn't implemented on hardware yet; global frequency plus per-board override is the proven path.
- **🧪 Army / clustering** — TNA-OS units can team up on your LAN: a **General** aggregates shares from  **Soldier** units and ESP devices (and proxies plain Bitmain miners), while Soldiers hash locally and
  submit upstream. Functional over wired Ethernet, still maturing.
- **🧪 Solar integration** — mine straight from your solar/battery setup, with PV, battery and charger   telemetry (VE.Direct or Modbus-TCP) built into the dashboard. The plumbing is in; live polling is   still being wired up.
- **🧪 Heat capture** — put your miner's waste heat to work (space and water heating), with exhaust and loop-temperature monitoring and a savings estimate. Early days.

The 🧪 features are improving fast — treat them as experimental.

---

## Resources

- **⚡ Flash your miner:** [TNA Flasher](https://lnbits.molonlabe.holdings/tnaflasher/public)
- **🔌 API reference:** [API.md](API.md)

---

## Mission & support

The mission is to liberate Bitcoin infrastructure from corporate dependencies through energy
independence, sovereign communications, and experimental technology.

If you find TNA-OS useful and would like to support continued research and development, please
consider supporting:

**👉 [molonlabe.holdings/#funding](https://www.molonlabe.holdings/#funding)**

Every sat helps. ⚡

---


**TNA-OS — your miner, your rules.**
