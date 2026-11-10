# K2 Plus Z-Offset by Filament & Build Plate

A Klipper macro for the Creality K2 Plus that sets the Z offset at the start of every print based on the **filament type** and **build plate** selected in OrcaSlicer.

## Install

1. In Fluidd → **Configuration**, upload `k2_zoffset.cfg`.
2. Add this line near the top of `printer.cfg`, then restart Klipper:
   ```
   [include k2_zoffset.cfg]
   ```
3. In OrcaSlicer → Printer settings → **Machine G-code** → *Machine start G-code*, add this line **after** `START_PRINT`, `T[...]` and `M109`, just before the purge line:
   ```
   SET_ZOFFSET_BY_MATERIAL FILAMENT="{filament_type[initial_no_support_extruder]}" PLATE="{curr_bed_type}"
   ```
   It must come after `START_PRINT`, because Creality's stock `START_PRINT` runs `SET_GCODE_OFFSET Z=0`.

## Configure

Edit the two tables at the top of the macro's `gcode:` section. Values are in mm. **Positive = nozzle higher (less squish).**

- **Filament table:** matched on the start of the slicer's filament type, with the longest match winning (`PETG-CF` uses `PETG`).
- **Plate table:** an exact match on the OrcaSlicer bed type name. Give each physical plate its own OrcaSlicer bed type:

| Physical plate | OrcaSlicer bed type |
|---|---|
| Stock textured PEI | Textured PEI Plate |
| Epoxy resin (smooth) | Smooth High Temp Plate |
| PEO side | Engineering Plate |
| PET side | Textured Cool Plate (or Smooth Cool Plate) |
| Holographic | Cool Plate (SuperTack) |

Set per-plate bed temperatures in each filament profile (Filament → Bed temperature).

Optional: `BASE=0.03` overrides the baseline offset for one print. The final offset is limited to `min_offset` to `max_offset` (−0.10 to +0.40 mm by default).

## Check it

At print start, the Fluidd console shows something like:
```
Z offset: filament 'PETG' -> PETG (+0.025), plate 'textured pei plate' -> textured pei plate (+0.000), base +0.000 => 0.025
```
`-> none` means no table entry matched. The message shows the exact name the slicer sent.

## After firmware updates

Creality firmware updates delete custom `.cfg` files but leave the `[include]` line in `printer.cfg`, which causes an XS3002 error. Re-upload `k2_zoffset.cfg` (or comment out the include) and restart.
