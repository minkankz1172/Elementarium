# Making mods for Elementarium

A mod is a single `.json` file. It is **only data**: it can add elements and add
reactions, but it can never run code, so installing a mod from someone else is safe.

## Installing a mod

Press **Add mods** (next to Changelog), then either

* click **Choose file...**,
* paste the JSON into the box and press **Install pasted mod**, or
* drop the `.json` file anywhere on the game.

Mods are remembered in your browser. In the list you can turn a mod **On/Off**,
**Export** it (to share it) or **Remove** it. Turning a mod off removes its
elements from the world and the toolbar; turning it on brings them back.
Installing a mod with the same name again updates it.

Press **Download template** to get a small working mod (`example-mod.json`) to start from.

## The file

```json
{
  "mod": "My Pack",
  "version": "1.0",
  "author": "You",
  "description": "What it does.",
  "elements": [ ... ],
  "patches":  [ ... ]
}
```

Only `mod` (the name) is required, plus at least one element or patch.

## Elements

```json
{
  "key": "sodium",
  "name": "Sodium",
  "lang": { "es": "Sodio", "zh": "钠" },
  "category": "solids",
  "movement": "solid",
  "color": [205, 208, 214],
  "density": 970,
  "reactions": [
    { "with": "water", "chance": 0.35, "to": "fire", "other": "hydrogen", "heat": 400 }
  ]
}
```

| Field | Meaning |
|---|---|
| `key` | Unique id: 2-32 lowercase letters, digits or `_`, starting with a letter. Cannot clash with the game or another mod. **Required.** |
| `name` | Name shown in the game (1-32 characters). **Required.** |
| `lang` | Optional names in other languages, e.g. `{"es":"Sodio","zh":"钠"}`. |
| `category` | Tab: `powders`, `solids`, `liquids`, `gases`, `life`, `food`, `phase`, `specials` or `mods` (default). |
| `movement` | `solid`, `powder`, `liquid`, `gas` or `clump` (a powder that does not slide). Default `solid`. |
| `color` | `[r,g,b]` (0-255, gets a little grain) or `{"from":[r,g,b],"to":[r,g,b],"animated":true}` (blends between two colors, shimmering if animated). |
| `swatch` | Optional `"#rrggbb"` for the toolbar button. |
| `density` | 0.001-30000. Heavier things sink through lighter liquids and gases. |
| `heatConduct` | 0-3. How fast heat flows through it (default 0.33). |
| `electroConduct` | 0-1. How well it carries electricity (wire is 1). |
| `acidr` | 0-3. How fast acid dissolves it. 0 or missing = acid cannot touch it. |
| `viscosity` | 0-300. Liquids only: higher is thicker and slower. |
| `rise`, `side`, `sink` | 0-1. Gases only: chance per tick to rise, drift sideways or sink. |
| `spray` | 0-1. When painting, chance that each cell of the brush is filled (lower = sparser). |
| `startMin`, `startMax` | Temperature (-273 to 9999) the element is pushed to when it is created (a minimum / a maximum). |
| `lifetime` | `[min,max]` ticks the cell lives. Used together with `decay`. |
| `decay` | `true`, or `{"below":T,"into":"key","chance":0-1}`: when `lifetime` runs out (and the temperature is at or below `below`) the cell becomes `into`, otherwise it disappears. |
| `burn` | `{"at","chance","heat","spread","water","flames","speed","into","emit"}`: ignites above temperature `at` with `chance` per tick, burns at `heat` degrees, sets touching flammable things alight with chance `spread`, is put out by water with chance `water`, throws flames with chance `flames`, burns down with chance `speed` per tick, then becomes `into` and releases `emit`. |
| `flammable` | `true` lets burning neighbors and fire ignite it. |
| `explosive` | `{"at":200,"radius":8,"heat":1200}`: explodes when hotter than `at` or touched by fire. Radius 1-20. |
| `corrosive` | Key of the element it turns into when it uses itself up dissolving things (acid-like). |
| `rustRate`, `rustInto`, `rusts`, `rustUses` | Rust mechanism: a solid with `rustRate` turns into `rustInto` next to a liquid/gas with `rusts` (0-1); `rustUses` makes that neighbor get used up. |
| `radiate` | -5 to 5. Heats (+) or cools (-) its surroundings every tick, like Heater and Cooler. |
| `insulator`, `unbreakable` | `true`: blocks heat; survives explosions. |
| `reactions` | List of reactions (below). Up to 24. |

Any field starting with `_` (for example `"_note": "..."`) is ignored, so you can leave comments.

## Reactions

Each tick, every cell tries its reactions **in order**; the first one that fires
(and has a `to`) ends the turn for that cell.

| Field | Meaning |
|---|---|
| `with` | An element that must be touching this cell. |
| `to` | This cell becomes this element. |
| `other` | The touching `with` cell becomes this element (`"air"` deletes it). Needs `with`. |
| `emit` | After changing, spawns this element in a free neighboring cell. Needs `to`. |
| `chance` | 0-1 chance per tick. |
| `above`, `below` | This cell's temperature must be higher / lower than this. |
| `heat` | Adds this many degrees to the cell when it fires (can be negative). |
| `warm` | Raises the temperature of touching cells to at least this. |
| `setT`, `capT` | After changing: set the temperature, or cap it at most. |
| `life`, `lifeBelow`, `keepLife` | Set the new cell's life `[min,max]`; only react while `life` is below a number; keep the old life value. |
| `rest` | Only react while the cell is resting on something (not falling). |
| `top` | Only react in the top N rows. |

A reaction needs at least one of `to`, `other`, `heat` or `warm`.

## Patches: changing existing elements

A patch adds reactions to an element that already exists (from the game or another mod):

```json
"patches": [
  { "element": "magma",
    "reactions": [ { "with": "glow_slime", "chance": 0.3, "to": "obsidian", "other": "smoke" } ] }
]
```

Patch reactions are added after the element's own reactions. Add `"first": true`
to put them before instead. When a mod is turned off its patches are removed again.
Patches can only add reactions, they cannot delete or change existing ones.

## Limits

* 100 elements per mod, 255 elements in total (game + all mods).
* 300 KB per mod, 24 reactions per element, 40 patches per mod.

If something is wrong, the panel lists every problem with its location,
for example `elements[2].color: [r,g,b] (0-255) ...`, and nothing is installed.

## The mod library

The **Library** tab of the Add mods panel is a list of mods picked by the author of the
game. Players press **Install** and the mod is added; from then on it lives in **My mods**
like any other mod (turn off, export, remove). Each entry can carry tags:

| Tag | Shown as | Use it for |
|---|---|---|
| `new` | green **New** | A mod that was just added. |
| `hot` | orange **Popular** | A mod people love. These are listed first. |
| `warning` | red **Warning** | A mod that can break the game or ruin a world. Players must confirm ("Install anyway?"). |
| `outdated` | grey **Outdated** | A mod made for an older version. Dimmed, listed last, must be confirmed. |

A short `note` from the author is shown under the description. Without a note, `warning`
and `outdated` entries show a default sentence.

### Where the library lives (GitHub)

The game reads the list from `library.json`, which sits next to the game file in your
GitHub repository (GitHub Pages). **Only people who can commit to your repository can
change the library**, so players can never change it or add a mod to it. If `library.json`
cannot be read (for example when the game is opened somewhere else) the game shows a small
built-in list instead.

```
index.html (or Elementarium.html)   the game
library.json                        the list: titles, tags, notes
mods/kitchen-chemistry.json         one file per mod
mods/example-pack.json
mods/hungry-goo.json
```

Each entry in `library.json` looks like this (the zip tool below writes it for you, and `tags`, `note` and `added` are optional):

```json
{ "id": "kitchen-chemistry", "file": "mods/kitchen-chemistry.json",
  "title": "Kitchen Chemistry", "author": "Elementarium", "version": "1.0",
  "description": "Baking soda and vinegar make a fizzy volcano.",
  "added": "2026-11-04", "tags": ["hot"], "note": "" }
```

`file` must be `mods/<name>.json`. `id` must be the mod's `mod` name written in lowercase
with dashes ("Kitchen Chemistry" becomes `kitchen-chemistry`).

### Managing the library (author)

Open the game with `#admin` at the end of the address, for example
`https://you.github.io/repo/#admin`. In the Library tab you now see author tools. They only
edit a copy in your own browser, nothing changes for players yet:

1. **Check a mod first.** Install the mod a player sent you (My mods, Choose file / paste) and
   play with it. If it is broken, just remove it, nobody else is affected.
2. **Add it.** In My mods press **Add to library** on that mod.
3. **Tag it.** In the Library tab, toggle `new`, `hot`, `warning` or `outdated`, write a note,
   or press **Remove from library** for mods you no longer want to list.
4. **Download library update (.zip)**. Unzip it.
5. **Upload to GitHub:** open your repository, press **Add file, Upload files**, drag in
   `library.json` and the `mods` folder, and commit. Players see the new list the next
   time they open the game (GitHub Pages can take a minute or two to update).

The zip only contains `library.json` plus the mod files that are new or changed.
To change only a tag or a note you can also edit `library.json` directly on github.com.
If you update a mod, raise its `version` so players who installed the old one get an
**Update** button.

The `#admin` address is not a secret and gives no power by itself: it cannot write to GitHub.
What protects the library is that only you can commit to the repository.

Players send you their mods with the **Send your mod to the author** button, which opens
your feedback form.
