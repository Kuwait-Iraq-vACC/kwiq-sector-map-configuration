# KWIQ sector map configuration

Staffing rules for the ACC Route Map Plotter in [KWIQ NavTools](https://github.com/Kuwait-Iraq-vACC/kwiq-navtools): which positions exist, and who owns each sector group in which order. The idea comes from the ESAA sector map ([ottotuhkunen/esaa-sector-map-configuration](https://github.com/ottotuhkunen/esaa-sector-map-configuration)), without presets.

NavTools reads `config.json` from branch `main` at runtime, through its server cache, which refreshes hourly. Admins can also press **Refresh from GitHub now** under Admin → Data sources. A change merged here shows on the site within the hour, with no site deploy.

Sector **shapes** are not in this repo. They come automatically from the ESE in the GNG packages uploaded to NavTools each AIRAC. This file only names the ESE sectors.

## config.json

```json
{
  "controllerList": [
    {"name": "ORBB", "fir": "ORBB", "id": "B", "type": "CTR"},
    {"name": "BN", "fir": "ORBB", "id": "BN", "type": "CTR", "color": "#4e79a7"},
    {"name": "KL", "fir": "OKAC", "id": "K", "type": "CTR"},
    {"name": "BGD_APP", "fir": "ORBB", "id": "BGD", "type": "APP"}
  ],
  "general": ["ORBB", "OKAC"],
  "buttons": {"ORBB": ["BN", "BNU", "BS", "BSU", "BGD_APP"], "OKAC": ["KL", "KU"]},
  "sectorsOwnership": {"ORBB-N-L": ["BN", "ORBB"], "BAGHDAD-TMA-2": ["BGD_APP"]},
  "groupTypes": {"ORBB-N-L": "CTR", "BAGHDAD-TMA-2": "APP"},
  "connectGroupWithRealSectors": {"ORBB-N-L": ["BAGHDAD-ACC-N-L"]}
}
```

| Key | Meaning |
|---|---|
| `controllerList` | Every position that can be staffed. `name` is used everywhere else in this file and on the buttons. `fir` and `id` are the FIR and the position ID in that FIR's ESE `[POSITIONS]`, and NavTools checks them against the ESE. `color` is optional (`#rrggbb`); without it NavTools picks a muted colour. |
| `general` | The bandbox buttons (the GENERAL row). |
| `buttons` | Button order per FIR. Sector positions come first; `*_APP` / `*_DEP` positions go on their own rows. |
| `type` (per controller) | `CTR`, `APP` or `TWR`: the position's level of service. |
| `groupTypes` | `CTR`, `APP` or `TWR` for each group: ACC sectors are `CTR`, TMA/CTA/approach volumes `APP`, aerodrome zones `TWR`. |
| `sectorsOwnership` | For each sector group, the positions that can own it, in priority order. The group is staffed by the **first position in the list that is online**, and drawn in that position's colour. |
| `connectGroupWithRealSectors` | For each group, the ESE `SECTOR` names (the middle part of `SECTOR:FIR·NAME·LOW·HIGH`) that make up the group. |

### Levels of service

There is **no top-down coverage across levels of service**:
- CTR positions only cover `CTR` groups (ACC sectors), never TMAs or aerodrome zones, even if the ESE OWNER line lists them as a fallback.
- APP positions only cover `APP` groups, never aerodrome zones.

Within a level, the ordered fallback stays: BN covers BNU when BNU is offline, and the bandbox covers all its ACC sectors. A group with no own-type owner online is drawn as an outline only. NavTools ignores (and reports) an owner whose type doesn't match the group, so only list positions of the group's type.

### KL and KU

There is no separate KL position in the ESE. `KL` is `OKAC_CTR` (ID `K`) limited to the volumes up to FL350, and `KU` is `OKAC_U_CTR`. So the groups up to FL350 list `["KL", "OKAC"]`, and the upper group lists `["KU", "OKAC"]`. `OKAC` is the bandbox, which is also `K` and covers everything. With KL on and KU off, the upper volumes stay unstaffed.

## Checks

NavTools compares this file with the uploaded ESE and lists any drift under Admin → GNG packages → Sectorfile checks:
- a name in `connectGroupWithRealSectors` that isn't an ESE sector any more;
- an ESE sector of ORBB or OKAC that isn't in any group;
- an ownership order that differs from the ESE `OWNER` line;
- a `controllerList` entry whose ID isn't in the ESE `POSITIONS`.

Admins can upload a local `config.json` there to test a change before committing it (Admin → Data sources → override).

## Regenerating

**Generate config.json from the ESE** on the same admin page (or `python -m navtools.sectorconfig` in NavTools) writes a fresh file from the current OWNER lines. Use it to see what changed after an AIRAC, then copy the parts you need. Don't overwrite your hand edits.
