# JLCPCB Order Specifications — FSAE Display Boards

Use these settings when ordering **fsae_pcb** (Main PCB) and **fsae_daughterboard** (Daughter Board) from JLCPCB.

---

## 1. Main PCB (fsae_pcb) — Raspberry Pi HAT

| Parameter | Setting |
|-----------|--------|
| **Board name** | fsae_pcb (Main PCB / Pi Shield) |
| **Dimensions** | 65 mm × 59.5 mm (from current Edge.Cuts) |
| **Layers** | 2 (double-sided) |
| **Board thickness** | 1.6 mm |
| **Material** | FR-4 TG 130–140 |
| **Copper weight** | 1 oz (35 µm) inner and outer |
| **Min. track / spacing** | 0.15 mm (6 mil) or per gerber |
| **Min. hole size** | 0.3 mm |
| **Solder mask** | Green (or your choice) |
| **Silkscreen** | White |
| **Surface finish** | HASL lead-free (or ENIG if desired) |
| **Gold fingers** | No |
| **Castellated holes** | No |
| **V-cut / scoring** | No (single board) |
| **Flying probe test** | Optional (recommended for first run) |
| **Assembly** | SMD + THT available; specify top side only (all parts on top). |

### Files to upload (Main PCB)

- **Gerbers:** Zip the folder `fsae_pcb/gerbers/` (include all .gbr, .drl, .gbrjob).
- **BOM:** `fsae_pcb/fsae_pcb_BOM_JLCPCB.csv`
- **CPL (placement):** `fsae_pcb/fsae_pcb_CPL_JLCPCB.csv`

---

## 2. Daughter Board (fsae_daughterboard)

| Parameter | Setting |
|-----------|--------|
| **Board name** | fsae_daughterboard (Button/LED panel) |
| **Dimensions** | 38 mm × 40 mm |
| **Layers** | 2 (double-sided) |
| **Board thickness** | 1.6 mm |
| **Material** | FR-4 TG 130–140 |
| **Copper weight** | 1 oz (35 µm) |
| **Min. track / spacing** | 0.15 mm (6 mil) or per gerber |
| **Min. hole size** | 0.3 mm |
| **Solder mask** | Green (or your choice) |
| **Silkscreen** | White |
| **Surface finish** | HASL lead-free (or ENIG) |
| **Assembly** | **Both sides** — SMD on bottom (B.Cu), THT on top (F.Cu). |

### Files to upload (Daughter Board)

- **Gerbers:** Zip the folder `fsae_daughterboard/gerbers/` (include all .gbr, .drl, .gbrjob).
- **BOM:** `fsae_daughterboard/fsae_daughterboard_BOM_JLCPCB.csv`
- **CPL (placement):** `fsae_daughterboard/fsae_daughterboard_CPL_JLCPCB.csv`

---

## 3. Assembly summary

| Board | SMD parts | THT parts | Assembly sides |
|-------|-----------|-----------|----------------|
| Main PCB | 27 | 12 | Top only |
| Daughter board | 9 | 7 | Top + Bottom |

---

## 4. Notes

- **Main PCB:** Raspberry Pi HAT; ensure mounting holes and J1 header match your HAT mechanical spec if you rely on it.
- **Daughter board:** J6 (Molex) and SMDs are on the **bottom** layer in CPL; LEDs and switches are on the **top**.
- JLCPCB part numbers in the BOMs are for basic/extended library; parts without a JLC number may need to be sourced separately or selected in JLCPCB’s part search.
