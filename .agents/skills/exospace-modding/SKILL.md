---
name: exospace-modding
description: "Use when creating, editing, testing, or publishing mods for eXoSpace Combat Engineer, including custom parts, weapons, projectile range/lifetime, effects, sounds, ships, factions, arenas, and Steam Workshop uploads."
argument-hint: "Describe the eXoSpace mod or gameplay change to create"
user-invocable: true
disable-model-invocation: false
---

# eXoSpace Modding

## Scope

Use the game's supported loose-file mod system. Do not modify the base `game_data/parts.json` for a distributable mod. Create a mod under `game_data/mods/<mod-id>/` and override only the definitions that need to change.

Official documentation: [eXoSpace modding guide](https://exospace-combat-engineer.com/modding)

## Game Paths

The Steam install contains:

```text
eXoSpace/
  exospace.exe
  game_data/
    settings.json
    parts.json
    effects.json
    mods/
```

On Linux, the Steam path is commonly:

```text
~/.local/share/Steam/steamapps/common/eXoSpace/game_data/
```

Confirm the actual install path before editing files. Preserve user changes and make a backup of any file before changing it.

## Enable Modder Tools

Edit `game_data/settings.json` and add this property before the final closing brace:

```json
"modders_tools": true
```

Maintain valid JSON, including the comma before this property when another property precedes it. The game uses a separate modding save location when these tools are enabled, reducing the risk to normal saves.

## Create a Mod

Create a directory such as `game_data/mods/longer_projectiles/` with:

```text
longer_projectiles/
  description.json
  preview.png
  parts.json
```

Use a stable lowercase ID and a clear description:

```json
{
  "id": "longer_projectiles",
  "name": "Longer Projectile Range",
  "description": "Increases projectile range and lifetime.",
  "version": 1
}
```

`preview.png` is used as the Workshop preview. The mod may also contain:

- `parts/` for part images
- `projectiles/` for projectile images
- `ships/` for ship designs
- `sounds/` for `.wav` files
- `effects.json` for particle effects
- `parts.json` for part definitions

Use the in-game modder tools for editing parts and particle effects where possible.

## Custom Sprite Color Channels

eXoSpace custom part and weapon textures use channel-coded color data rather than
ordinary display colors:

- **Red channel:** the main sprite mask, multiplied by the part's faction color.
- **Blue channel:** emissive/glow mask, multiplied by the configured glow color.
- **Green channel:** currently unused by the documented shader.
- **Alpha:** sprite transparency.

Keep connector-edge pixels opaque where the part meets adjacent blocks. Avoid
placing transparent padding over connector edges, since it makes the part appear
visually disconnected even when the connector coordinates are correct. For custom
sprites, author the mask channels deliberately; a normal full-color PNG may appear
black or excessively faction-tinted in-game.

## Override Built-in Weapons

To change an existing weapon, copy its complete definition from the matching object in `game_data/parts.json` into the mod's `parts.json`. The official guide supports overriding built-in parts this way.

Weapon definitions commonly include:

```json
"weapon": {
  "velocity": 2000.0,
  "range": 1500.0,
  "lifetime": 10.0
}
```

For longer projectile travel:

1. Identify the built-in weapon entries to change.
2. Copy those part definitions into the mod's `parts.json`.
3. Increase `range` and, when needed, `lifetime`.
4. Keep `velocity` unchanged unless changing projectile speed is intentional.
5. Test both player and enemy weapons. An override may affect every use of that built-in part.
6. Check beams, mines, and special weapons separately; zero-valued range or lifetime fields may have special behavior.

Prefer targeted overrides over changing every weapon unless the mod explicitly intends global balance changes. Do not assume that changing the base `parts.json` is Workshop-compatible.

## Load and Test

1. Start the game after creating the mod directory.
2. Open the Mods screen.
3. Enable the mod and adjust its load order if necessary.
4. Use the modder tools and sandbox to test the change.
5. Check for missing assets, duplicate IDs, invalid JSON, and conflicts with other mods.
6. Test a fresh sandbox scenario before using the mod in an existing assignment or save.

When diagnosing a failure, first inspect the game's mod screen and logs, then validate JSON with a parser such as `jq`:

```bash
jq empty game_data/mods/<mod-id>/description.json
jq empty game_data/mods/<mod-id>/parts.json
```

## Publish to Steam Workshop

Use the in-game modder tools:

1. Open the modder tools for the mod.
2. Select **Publish mod on Steam Workshop**.
3. Set the Workshop name and description.
4. Publish the mod.
5. Accept Steam's Workshop legal agreement if prompted.
6. Use the same dialog later to update the existing Workshop item.

The game stores the Workshop item reference in the mod's `description.json`. Do not use SteamCMD for the normal creator workflow; SteamCMD is a generic Steamworks testing path, while eXoSpace provides its own publisher UI.

Workshop items should contain the mod's supported files, not a replacement copy of the entire installed `game_data` directory. Document required load order and known conflicts in the Workshop description.

## Supported Content Boundaries

Supported through the documented system:

- New or overridden parts and weapons
- Projectile, part, and effect graphics
- Particle effects
- Weapon sounds
- Ship designs
- Factions and arenas through their in-game editors

Do not claim support for shader replacement, executable patches, or unsupported systems such as perks unless the current official documentation confirms it. The developer has stated that perk modding was not supported in the referenced discussion.

## Safety and Compatibility

- Never overwrite the original game data when creating a Workshop mod.
- Back up files before enabling modder tools or experimenting with overrides.
- Steam updates can change the built-in schema and break overrides.
- Existing ships, assignments, or saves may warn about missing or changed mods.
- Record the game version and mod version in the Workshop description.
- Keep IDs unique and avoid renaming an existing published mod ID.
- Verify that the mod loads with no other mods enabled before investigating mod conflicts.

## Official References

- [Modding guide](https://exospace-combat-engineer.com/modding)
- [Developer's modding announcement](https://steamcommunity.com/app/2876200/discussions/0/595159519773917682/)
- [Developer example: SALPEN Launcher Mod](https://steamcommunity.com/sharedfiles/filedetails/?id=3566666032)
- [eXoSpace Workshop](https://steamcommunity.com/app/2876200/workshop/)
