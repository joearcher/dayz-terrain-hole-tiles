# Terrain Hole Tiles

A single-page tool for DayZ modders that converts world positions into terrain hole tile coordinates and writes the `Holes` config for a mod.

**Use it:** https://tiles.bgom.dev/

## What it does

DayZ 1.30 added terrain holes: tiles of the terrain that are removed so players can walk underground. A world's config lists them as tile coordinates:

```cpp
class Holes
{
    class MyGroup
    {
        tiles[] =
        {
            { 1417, 3415 },
            { 1418, 3415 }
        };
    };
};
```

A tile is one heightmap cell:

```
tile = { floor(X / grid size), floor(Z / grid size) }
```

It's counted from the south-west corner of the map, X first, then Z (north). Tile `{x, z}` covers `x·g` to `(x+1)·g` metres east and `z·g` to `(z+1)·g` metres north.

The page:

- converts a single position straight away: type the X and Z from the game, or paste a whole `X, Y, Z` position, and it shows the tile, its bounds and nearby edges. From there you can copy it as `{ x, z }`, add it to your list or show it on the grid;
- reads lists of positions as DayZ prints them (`X, Y, Z`), as `X Z` pairs, or as tiles typed directly. You can paste an existing `tiles[]` block back in;
- adds the tile under each position, every tile within a radius, or every tile in the box between the positions;
- shows the tiles on a north-up grid with the map's vanilla holes and the edge of the terrain;
- lets you paint tiles in or erase them by clicking or dragging. Scroll to zoom; pan with Move view, Shift-drag or right-drag; Fit, Undo (Ctrl+Z) and Reset edits. The keyboard works too: arrow keys, Space, + / − and F;
- keeps separate positions and edits for each map, with an example to start from;
- warns when a group name would replace a vanilla group;
- writes either the `Holes` block alone or a complete mod `config.cpp`, to copy or download.

## Grid sizes

| Map | Grid size | Tiles | Source |
|---|---|---|---|
| Nasdara | 4 m | 4096 × 4096 | Measured (see below) |
| Sakhal | 3.75 m | 4096 × 4096 | [BI wiki](https://community.bistudio.com/wiki/DayZ:Terrain_holes) |
| Livonia (Enoch) | 6.25 m | 2048 × 2048 | BI wiki |
| Chernarus | 7.5 m | 2048 × 2048 | BI wiki |

There's also a custom option for other terrains.

### How Nasdara was measured

The wiki doesn't list Nasdara, and its world config ships encrypted. These measurements came from a probe mod on a 1.30 dedicated server:

- `GetWorldSize()` returns 16,384 m.
- Sampling `SurfaceY` every 12.5 cm shows the terrain's slope changing on every 4 m line, along both axes.
- `SurfaceIsHole` finds every existing hole's edges on 4 m lines. For example, the irrigation shaft at 5670, 13661 is a hole from x 5668 to 5672 and z 13660 to 13664: tile `{ 1417, 3415 }`.
- A scan of every 4 m tile found 556 vanilla hole tiles. Those are the hatched tiles on the grid. Nasdara's config has 33 `Holes` groups.

### Adding holes from a mod

A mod can add its own group to a vanilla world. On a 1.30 Nasdara server, this config left all the vanilla groups in place and turned exactly the listed tiles into holes, and it has since been checked in game:

```cpp
class CfgPatches
{
    class MyMod_TerrainHoles
    {
        units[] = {};
        weapons[] = {};
        requiredVersion = 0.1;
        requiredAddons[] = { "DZ_Data", "DZ_Worlds_Nasdara" };
    };
};

class CfgWorlds
{
    class CAWorld;
    class nasdara: CAWorld
    {
        class Holes
        {
            class MyMod_Bunker
            {
                tiles[] = { { 3326, 3100 }, { 3327, 3100 } };
            };
        };
    };
};
```

- Give your groups a unique prefix: a group with the same name as a vanilla one replaces it.
- Load the mod on the server and on clients.
- The engine reads holes when the map loads, so they can't be changed from script or a settings file at runtime.

## Notes

`index.html` is the whole tool, with no build step and no dependencies apart from Google Fonts. Not affiliated with Bohemia Interactive.
