# PrintCal — Ender calibration calculator for Klipper & Marlin

PrintCal turns 3D-printer test-print measurements into the exact settings to change. It covers PID tuning commands, extruder `rotation_distance` / e-steps, pressure advance, input shaper, flow ratio, max volumetric speed, temperature and retraction towers, skew, shrinkage and X/Y/Z axis steps. It has presets for 20 Creality Ender printers and 19 extruders, including direct-drive upgrades (Sprite SE/Pro, Micro Swiss, Bondtech, E3D, Orbiter, Sherpa Mini, Voron Clockwork 2, H2). Every result is range-checked against your hardware before you can copy it.

It's one `index.html` file: no build step, no server, no tracking.

**Live:** https://sleeeeeeeeeeeeeep-cmyl.github.io/printcal/

---

## Using it

1. **Describe your printer** under *Your printer*: model, firmware, extruder, hotend, nozzle. Starting values, the commands shown (Klipper or Marlin) and every safety limit follow from this.
2. **Calibrate in order:** PID tuning (only after a hotend, heater or fan change) → extruder → pressure advance → flow ratio → max volumetric speed, then towers, input shaper, skew and shrinkage as needed. Each step assumes the earlier ones are right. *Axis steps* is a reference card for X/Y/Z values after a board swap or rebuild.
3. **Type your measurements over the example numbers.** A result built from example numbers is never added to *All changes*.
4. **Test, then save.** Klipper results give a live command (forgotten on restart) and the line to make permanent in `printer.cfg`. Marlin results end with `M500`.
5. **Apply everything** from *All changes*.

What the result markers mean:

| Marker | Meaning |
|---|---|
| tinted input | filled in from your preset; replace it with your printer's real value |
| **OK** | inside the expected range for your setup |
| **!** | shown, but unusual; read the warning before applying |
| **✕** | hidden because it's outside a safe range, usually a measuring or mechanical problem; the override button shows it anyway |

Settings are saved in your browser's local storage only. *Reset everything* clears them.

---

## Publishing your own copy

1. Create a GitHub repository (e.g. `printcal`) and add `index.html` and this README.
2. In `index.html`, replace `sleeeeeeeeeeeeeep-cmyl` with your own GitHub username, and `printcal` with your repository name if it differs (three places: canonical link, Open Graph URL and structured data).
3. Repository → **Settings → Pages** → *Deploy from a branch* → `main` / root → Save.
4. The site appears at `https://<username>.github.io/<repository>/` within a minute or two.

---

## Customizing

Everything you'd want to change is in one `<script>` block near the end of `index.html`, marked **`CUSTOMIZE HERE`**:

| Name | What it is |
|---|---|
| `DEFAULT_PRINTER` | id of the printer selected on a first visit |
| `LIMITS` | the safety thresholds (warn / block) |
| `PRINTERS` | one line per printer |
| `EXTRUDERS` | one line per extruder |

Edit, save, reload. You don't need to touch the code below that block.

### Add a printer

Copy a similar line in `PRINTERS`, change it, and keep the commas between lines. Only `id` and `name` are required; anything left out uses the default shown.

```js
{"id": "my_ender3", "name": "My Ender 3 (Sherpa Mini)", "fam": "My printers", "cfg": "https://example.com/printer.cfg", "rd": 22.67895, "gear": "50:10", "maxT": 285, "accel": 4000, "vel": 350, "meo": 120, "z": 8, "bedT": 110, "drive": "direct", "hot": "allmetal", "fw": "klipper", "note": "Shown under the setup when this printer is picked."},
```

| Field | Meaning | Default |
|---|---|---|
| `id` | unique; letters, digits, `-`, `_` | required |
| `name` | dropdown label | required |
| `fam` | dropdown group | `"Other"` |
| `cfg` | Klipper config file name in `Klipper3d/klipper/config/`, or a full `https://` link | `null` (none) |
| `borrow` | when `cfg` is null: which printer the values came from (shown in the reference table) | `null` |
| `rd` | stock extruder `rotation_distance` | `null` (user enters their own) |
| `gear` | stock `gear_ratio`, `"a:b"` or `"a:b, c:d"` (needs `rd`) | `null` |
| `maxT` | hotend `max_temp`, °C | `null` |
| `accel` | `max_accel`, mm/s² | `null` |
| `vel` | `max_velocity`, mm/s | `null` |
| `meo` | `max_extrude_only_distance`, mm (Klipper) | `null` → Klipper's default 50 |
| `z` | Z `rotation_distance` (the *Axis steps* card compares against it) | `null` |
| `bedT` | heated-bed `max_temp`, °C (caps the bed PID target when it's below `BED_CAP_C`) | `null` |
| `ratedT` | maker's nozzle temperature rating, °C; used as the tower cap for an all-metal stock hotend when `maxT` is unknown | `null` |
| `spec` | `https://` link to the maker's spec page, shown when there's no `cfg` | `null` |
| `kin` | kinematics label | `"cartesian"` |
| `drive` | `"bowden"` or `"direct"` | `"bowden"` |
| `hot` | `"ptfe"` or `"allmetal"` | `"ptfe"` |
| `fw` | `"marlin"`, `"marlin212"`, `"klipper"` or `"crealityos"` | `"marlin"` |
| `note` | text shown in setup | none |

Numbers are written without quotes (`"rd": 34.406`, not `"rd": "34.406"`). Use `null` for unknown values rather than guessing; PrintCal then asks the user for their real value and skips the checks that need it.

Your printer appears in the dropdown and in the reference table automatically.

### Add an extruder

```js
{"id": "my_ext", "grp": "Mine", "name": "My geared extruder", "esteps": 500, "gear": "3:1", "defaultGear": true, "drive": "direct", "retractMax": 1.5, "src": "https://example.com/manual.pdf", "srcName": "Maker's manual"},
```

| Field | Meaning | Default |
|---|---|---|
| `id`, `name` | as above | required |
| `grp` | dropdown group | `"Other"` |
| `esteps` | maker's steps/mm at 16 microsteps on a 1.8° motor | `null` (user enters their own) |
| `drive` | `"bowden"`, `"direct"` or `"either"`; picking the extruder sets *Mounting* unless `"either"` | `"either"` |
| `gear` | Klipper `gear_ratio`, `"a:b"` or `"a:b, c:d"` (needs `esteps`) | `null` |
| `gearRd` | the `rotation_distance` to use **with** that gear ratio, if the maker publishes one | computed: 3200 × ratio ÷ esteps |
| `defaultGear` | `true` = tick the gear_ratio box when picked, because the maker's Klipper config uses it (needs `gear`) | `false` |
| `retractMax` | maker's retraction limit, mm; the retraction tower warns above it | none |
| `hot` | `"ptfe"` or `"allmetal"`: sets the *Hotend* field when picked, for kits that include their own hotend | none |
| `note` | text shown in setup | none |
| `src`, `srcName` | `https://` link and label for the reference table | none |

**Maker only gives Klipper values?** Convert to `esteps`:
`esteps = 200 × 16 × gear ratio ÷ rotation_distance`. For example, Sherpa Mini `rotation_distance 22.67895` with `gear_ratio 50:10`: 3200 × 5 ÷ 22.67895 = 705.5. Put the maker's own number in `gearRd` so it's used exactly.

**Don't remove** the extruder with `"id": "stock"`. It means "use the printer preset's extruder". If it's missing, PrintCal adds it back and tells you.

### Change the safety limits

Edit the numbers in `LIMITS`. The *Safety limits* table on the page always shows the values in use.

| Setting | Default | Effect |
|---|---|---|
| `EX_WARN` / `EX_BLOCK` | 0.10 / 0.25 | extruder correction above 10 % warns, above 25 % blocks |
| `NOMINAL_MISMATCH` | 0.25 | warns when the current value is more than 25 % from the preset |
| `PTFE_CAP_C` | 240 | temperature-tower cap for PTFE-lined hotends, °C |
| `OVERSHOOT_C` | 15 | cap for all-metal hotends is `max_temp` minus this |
| `PA_MAX` | 1.0 | pressure advance above this is blocked (Klipper's typical range tops out at 1.0) |
| `PA_DD_WARN` | 0.2 | pressure advance above this warns on direct drive |
| `IS_MIN` / `IS_LOW` / `IS_MAX` | 15 / 25 / 150 | input-shaper Hz: blocked outside MIN–MAX, warns below LOW |
| `RETRACT_DD_WARN` / `RETRACT_BOWDEN_WARN` | 2 / 7 | retraction-tower warnings, mm, when the extruder has no `retractMax` |
| `FLOW_LO` / `FLOW_HI` | 0.85 / 1.15 | flow ratio warning band |
| `SKEW_WARN_DEG` | 1 | skew warning, degrees out of square |
| `SCALE_WARN_PCT` / `XY_DIFF_WARN_PCT` | 3 / 0.3 | shrinkage warnings, % |
| `BED_CAP_C` | 110 | highest bed temperature PrintCal builds a PID command for |

Only the temperature cap, pressure-advance range and maker retraction limits come from documentation. The rest are PrintCal's own conservative choices. Each value must be a positive number. Pairs must stay in order (`EX_WARN` < `EX_BLOCK`, `IS_MIN` < `IS_LOW` < `IS_MAX`, `FLOW_LO` < 1 < `FLOW_HI`), or PrintCal reverts that pair to its defaults.

### When something's wrong

PrintCal never calculates with a broken preset. A bad entry is skipped, and a red box at the top of the page says which one and why, for example:

> PRINTERS line 20 ("My Ender"): drive must be "bowden" or "direct". Skipped.

Typical causes: a typo in a fixed value (`"Direct"` instead of `"direct"`), a gear written `42/12` instead of `42:12`, a number in quotes, a duplicate `id`.

A **syntax error** (a missing comma, quote or bracket) stops the whole block. The red box says so, and the browser console (F12) gives the line number. Also, never type a closing script tag inside any text in the block, because the browser ends the block there.

Renaming or removing an `id` is safe: visitors who had it selected are moved back to the default.

---

## How the maths works

| Calculator | Formula | Source |
|---|---|---|
| Klipper extruder | `new rotation_distance = old × actual ÷ requested` | [Klipper: Rotation distance](https://www.klipper3d.org/Rotation_Distance.html) |
| Marlin extruder | `new e-steps = old × requested ÷ actual` | same method, inverted |
| Pressure advance (Klipper tower) | `PA = START + height × FACTOR` (FACTOR .005 direct drive, .020 bowden) | [Klipper: Pressure advance](https://www.klipper3d.org/Pressure_Advance.html) |
| Pressure advance (Orca tower) | `PA = start + step × height` | [OrcaSlicer wiki](https://www.orcaslicer.com/wiki/calibration/pressure_advance_calib.html) |
| Input shaper (ringing tower) | `f = V × N ÷ D` | [Klipper: Resonance compensation](https://www.klipper3d.org/Resonance_Compensation.html) |
| Marlin M593 tower | `f = start + span × (layer − 2) ÷ layers` | [Marlin M593](https://marlinfw.org/docs/gcode/M593.html) |
| Flow ratio | 2-pass `old × (100 + mod) ÷ 100`; YOLO `old + mod` | [OrcaSlicer wiki](https://www.orcaslicer.com/wiki/calibration/flow_ratio_calib) |
| Max volumetric speed | `start + height × step`; line area `h × (w − h(1 − π/4))` | [OrcaSlicer wiki](https://www.orcaslicer.com/wiki/calibration/volumetric_speed_calib.html), [Flow.cpp](https://github.com/SoftFever/OrcaSlicer/blob/main/src/libslic3r/Flow.cpp) |
| Skew | Klipper's `calc_skew_factor` | [Klipper: Skew correction](https://www.klipper3d.org/Skew_Correction.html) |
| Axis steps | `rotation_distance = belt pitch × pulley teeth` or `screw pitch × thread starts`; `steps/mm = motor steps × microsteps ÷ rotation_distance` | [Klipper: Rotation distance](https://www.klipper3d.org/Rotation_Distance.html) |
| PID tuning | builds `PID_CALIBRATE HEATER=… TARGET=…` or `M303 E… S… C… U1`; no maths | [Klipper: Configuration checks](https://www.klipper3d.org/Config_checks.html), [Marlin M303](https://marlinfw.org/docs/gcode/M303.html) |

Printer presets come from [Klipper's official printer configs](https://github.com/Klipper3d/klipper/tree/master/config). The Ender 3 V3 family has no upstream config, so its speed, acceleration and nozzle rating come from [Creality's spec comparison](https://creality.com/compare/compare-ender-3-v3-series). Extruder values come from each maker's documentation, linked in the page's reference table.

Made by **Vidom**. Always back up your printer config before changing it.
