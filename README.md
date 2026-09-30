<p align="center"><img src="logo.png" width="128" alt="logo"></p>

# mod-transmog plugin

Builds [azerothcore/mod-transmog](https://github.com/azerothcore/mod-transmog) as a plugin for
[LonelyIceProject/azerothcore-wotlk](https://github.com/LonelyIceProject/azerothcore-wotlk), the way a package spec
builds a program from its original sources: the module is not forked, this repository only says how to build it.

Changes the look of equipped items to that of other items.

| File | Purpose |
|---|---|
| `plugin.json` | The plugin manifest; `source` pins the module's repository and commit. |
| `CMakeLists.txt` | Fetches the module at that commit, applies `patches/`, builds it with `AddPlugin` and lays out its `conf` and `data` folders with the plugin. |
| `plugin/plugin.cpp` | The plugin's entry point: the module's own script loader `Addmod_transmogScripts()`. |
| `settings.json` | The launcher's settings group: the module's main options from its `.conf.dist`, with how each one takes effect. |
| `patches/` | Changes the module needs as a plugin, applied with `git apply`. |

## Patches

None: the module builds and runs as a plugin unchanged.

## Build

The core built with `-DWITH_DYNAMIC_LINKING=ON` (a static core builds the plugin into worldserver instead):

```
cmake -S azerothcore-wotlk -B build -DWITH_DYNAMIC_LINKING=ON -DAC_PLUGIN_ABI=lonelyice-ac-2 ^
      "-DAC_PLUGIN_SOURCE_DIRS=<path>/mod-transmog-plugin"
cmake --build build --config RelWithDebInfo
```

The plugin is laid out in `bin/<config>/plugins/mod-transmog/`; copy that folder into the server's `plugins` folder
(`PluginsDir` in worldserver.conf). [LonelyIce](https://github.com/LonelyIceProject/lonelyice) builds and installs it
for you.

## Updating the module

Set `source.commit` in `plugin.json` to the new commit, raise `version`, delete `build/_deps/mod-transmog-*`
and configure again: the module is fetched and patched anew, and a patch that no longer applies stops the configure
step. To rework a patch, edit the checkout in `build/_deps/mod-transmog-src`, save `git diff` into
`patches/` and delete the `_deps` folders the same way.

## License

The module is licensed by its authors under AGPL-3.0; this repository (build files and patches) is under the same license, see [LICENSE](LICENSE).

### Blizzard Entertainment

World of Warcraft®, Warcraft®, Wrath of the Lich King® and Blizzard Entertainment® are trademarks or registered
trademarks of Blizzard Entertainment, Inc. in the U.S. and/or other countries.

The game and everything in it belong to Blizzard Entertainment, Inc.: the game client and its program files, data
files and archives, maps and terrain, models, textures, art, animations, interface, music, sounds, voices, texts,
names, lore, characters, creatures, spells, items, quests and every other part of the game. All of it remains
Blizzard's property wherever it appears, including the data a server extracts from your client on your own computer
(game tables, maps, collision and navigation data).

LonelyIce is an unofficial, non-commercial fan project. It is not affiliated with, endorsed, sponsored, approved or
supported by Blizzard Entertainment, Inc. Blizzard's names are used only to say which game client the project works
with.

This repository contains no files from the game client, and LonelyIce neither distributes nor downloads any. It works
only with a copy of the game you already own. Keep the data extracted from your client to yourself: it is Blizzard's
property and is not ours or yours to share.

The license above covers only the code and files of this project and the works it is based on. It grants no rights
to anything that belongs to Blizzard Entertainment, Inc. All other trademarks belong to their respective owners.
