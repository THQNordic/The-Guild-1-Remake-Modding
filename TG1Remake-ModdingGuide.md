# The Guild 1 Remake: Europa 1410 - Localization Modding Guide

**At this time, The Guild 1 Remake: Europa 1410 only supports localization mods. We may expand to support other mods in the future.**


## Overview

A localization mod replaces the game's texts with your own translation. It is a folder with one `mod.json` manifest and one or more CSV files, nothing else. No tools, no game files are modified, and the mod loads on top of the official localization.

What a localization mod can do:

- Translate the game into a language that Europa 1410 does not ship. The language appears in the settings under Options, Language with the name you give it.
- Improve an official translation. Every row in your CSV overrides that one text, every text you leave out stays official.
- Cover only part of the game. Texts you do not translate fall back to the official translation of that language, then to English.

What it cannot do yet: change texts that are not in the game's string tables (a few texts built into the interface code), add fonts, or replace voice-over. Players pick up your mod by subscribing to it on the Steam Workshop. The game loads subscribed items on start and when a subscription changes while it runs.

This guide is for translators with no modding experience. You need a text editor or a spreadsheet application, and a Steam account that owns the game for publishing.

## Before you start

You need three things: the text template, a language code, and a place to test.

**The text template.** All translatable texts live in the game's string tables, one CSV per topic (Actions, Buildings, Items, Politics, Notifications and so on, 31 tables with about 7,600 texts). Download the CSV templates from the StringTables folder of this repository. Each file has a `Key` column, the English `SourceString`, and an empty `Translation` column that you fill in.

**A language code.** The manifest names the language with a standard culture code, for example `it` for Italian, `nl` for Dutch, `pt-BR` for Brazilian Portuguese or `zh-Hant` for Traditional Chinese. Any code the game can resolve works. The code `ash` is reserved for internal use and is rejected. If the game already ships that language, your mod overrides it text by text.

**A place to test.** Local mods load from the Mods folder inside the game's installation:

```
<Steam library>\steamapps\common\<Europa 1410 install folder>\Europa1410\Mods\
```

Every direct subfolder of Mods is one mod. The folder name does not matter, the manifest inside it does. Create the folder if it does not exist.

**For publishing** you additionally need [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD), a Steam account that owns Europa 1410, and a one-time acceptance of the [Steam Workshop legal agreement](https://steamcommunity.com/sharedfiles/workshoplegalagreement).

## Mod folder structure

A mod is one folder with a `mod.json` manifest at its top level. Everything else is up to you. Refer to the example mod for more details.

```
ItalianTranslation\
    mod.json
    Localization\
        ST_Actions.csv
        ST_Buildings.csv
        ...
```

The manifest is a JSON text file, saved as UTF-8:

```json
{
    "FormatVersion": 1,
    "Id": "yourname.italian",
    "Name": "Italian Community Translation",
    "Author": "Your name",
    "Version": "1.0.0",
    "LastUpdated": "2026-10-29",
    "Type": "Localization",
    "Localization": {
        "Culture": "it",
        "DisplayName": "Community",
        "Files": []
    }
}
```

| Field | Required | Rules |
| --- | --- | --- |
| FormatVersion | yes | Always `1`. Other values are rejected. |
| Id | yes | Unique id of your mod, up to 64 characters: letters, digits, `.`, `-`, `_`. Use `yourname.language` so it never collides with another mod. |
| Name | yes | Mod name shown in the game's log, up to 128 characters. |
| Author, Version, LastUpdated | no | Free text for your own bookkeeping. |
| Type | yes | Always `Localization`. |
| Localization.Culture | yes | Culture code of the language, see above. |
| Localization.DisplayName | no | Up to 64 characters. Shown after the language in the settings dropdown, for example `italiano (Italian) - Community`. Leave it empty to show only the language name. |
| Localization.Files | no | List of CSV files relative to the mod folder, for example `["Localization/*.csv"]`. A wildcard matches files in that one folder. Empty list or missing field: every `.csv` in the mod folder and its subfolders loads. Files outside the mod folder are refused. |

Two mods with the same `Id` do not load together. A mod in the local Mods folder wins over a Workshop subscription with the same id, which lets you test a new version while still subscribed to the published one. The manifest may not exceed 64 KB.

## Writing the translation

The game reads plain CSV files: comma separated, UTF-8, first row is the header. Columns are found by name, so their order is free and extra columns are ignored. That is why the template files work as they are: fill in `Translation`, keep the rest.

| Column | Required | Meaning |
| --- | --- | --- |
| Key | yes | The text's key, exactly as in the template. |
| Translation | yes | Your text. An empty cell keeps the official text. |
| Namespace | no | Name of one string table, for example `Actions`. Only needed when the same key exists in several tables and you want different translations per table. Without it the row applies to every table that holds the key. |
| SourceString | no | The English original from the template. Ignored by the game, keep it for reference. |

A minimal file:

```csv
Key,SourceString,Translation
Action_DisplayName,Action,Azione
Action_DisplayName_plural,Actions,Azioni
Benefits_Office,Benefits of {office},Benefici di {office}
```

Rules that matter:

- **Keep placeholders exactly.** Anything in curly braces such as `{office}`, `{P_ACTPRO_C}...{##}` or `{Keyword_(...)}`, and tokens like `$Config|Field$`, is replaced by the game at runtime. Copy them unchanged into your translation, you may move them within the sentence.
- **Quote cells that contain commas, quotes or line breaks.** Wrap the cell in double quotes and double any quote inside: `"He said ""yes"", then left"`. A spreadsheet application does this for you on export.
- **Save as UTF-8.** Excel: File, Save As, type "CSV UTF-8 (Comma delimited)". LibreOffice: character set Unicode (UTF-8), field delimiter comma.
- **Leading and trailing spaces in `Translation` are kept.** Keys are trimmed.
- Rows with an unknown key are skipped with a warning in the log. Rows with a translation but no key are counted as malformed.
- Keys ending in `_plural` are the plural form of the key before them, translate both.
- Each CSV may be at most 10 MB. Split large translations across files, all files of a mod merge.
- Later files override earlier ones for the same key. Keep each key in one file to avoid surprises.

You do not need to translate everything before publishing. Start with the tables players see most, General, Actions, Buildings, Items and Notifications, and extend the mod over time.

## Testing locally

1. Copy your mod folder into `Europa1410\Mods\` inside the game installation (see Before you start).
2. Start the game. Mods load before the main menu appears.
3. Open Options, Language. Your language is listed as `<native name> (<English name>) - <DisplayName>`, for example `italiano (Italian) - Community`. For a language the game already ships, the official entry is replaced by yours.
4. Select it and apply. The texts switch at once, no restart needed.
5. Walk through the screens you translated. Texts you did not translate show the official translation of that language, or English if there is none.

After you change a CSV, restart the game to reload the mod.

The log tells you what was loaded. Open the newest file in `%LOCALAPPDATA%\Europa1410\Saved\Logs\` and search for `LogGuildMods`. The lines you want to see:

```
LogGuildMods: Mod discovery finished: 1 loaded (1 local, 0 workshop), 0 skipped
LogGuildMods:   yourname.italian 'Italian Community Translation' by Your name (Localization) from .../Mods/ItalianTranslation. Source: Local
LogGuildMods: Mod 'yourname.italian' (it): 1532 translations loaded, 0 unknown keys, 0 malformed rows.
```

A `Skipping mod folder` error names the reason the manifest was rejected, and every unknown key gets its own warning with the file name.

## Publishing to Steam Workshop

Uploads go through SteamCMD with a small description file. The Workshop item contains exactly your mod folder, so subscribers get the same `mod.json` and CSV layout you tested.

1. Install [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD) and log in once with your account, the one that owns Europa 1410.
2. Accept the [Workshop legal agreement](https://steamcommunity.com/sharedfiles/workshoplegalagreement) once. Without it the upload fails with an unhelpful error.
3. Prepare a preview image, square, 512 by 512 pixels, under 1 MB. Keep it outside the mod folder so it is not uploaded as mod content.
4. Save this file as `upload.vdf`, outside the mod folder, with your paths. Use forward slashes or doubled backslashes.

```
"workshopitem"
{
    "appid"            "2977260"
    "publishedfileid"  "0"
    "contentfolder"    "C:/Mods/ItalianTranslation"
    "previewfile"      "C:/Mods/ItalianTranslation_preview.png"
    "visibility"       "2"
    "title"            "Italian Community Translation"
    "description"      "Community Italian translation for Europa 1410."
    "changenote"       "Initial upload"
}
```

5. Run the upload:

```
steamcmd +login <your_steam_account> +workshop_build_item "C:/Mods/upload.vdf" +quit
```

6. Open the item page in Steam, set the tag Localization and the language, then switch visibility to public.

| VDF field | Meaning |
| --- | --- |
| appid | Europa 1410's Steam app id, always `2977260`. |
| publishedfileid | `0` creates a new item. SteamCMD rewrites the file with the real id after the upload. Keep the file, the id is what later updates use. |
| contentfolder | The folder that directly contains `mod.json`. Everything inside is uploaded. |
| previewfile | Thumbnail on the Workshop page. Required for a new item. |
| visibility | `0` public, `1` friends only, `2` private, `3` unlisted. Start private, check the page, then publish. |
| title, description | Shown on the Workshop page, editable there later. |
| changenote | Appears in the item's change history on each upload. |

**Updating a published mod.** Edit the CSVs, bump `Version` in `mod.json`, change the `changenote` and run the same command again. SteamCMD updates the item in place because the VDF now holds its id. Subscribers receive the update through Steam, and a running game reloads its mods a moment after the download finishes.

**What subscribers see.** Steam installs the item under `steamapps/workshop/content/2977260/<item id>/`. The game loads it like a local mod, including the language entry in the settings. Unsubscribing removes the language again, and an open game switches back to the official language right away.

## Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| Language not in the settings list | Manifest rejected, folder not scanned, or culture code wrong | Search the log for `Skipping mod folder`. The message names the field. Check that `mod.json` sits directly in the mod folder and `FormatVersion` is 1. |
| `Mod manifest file not valid json` | Syntax error in `mod.json` | Common causes: a missing comma, a trailing comma after the last field, curly quotes from a word processor. Validate the file with any JSON checker. |
| `Localization culture xx is not a known culture code` | Typo in `Culture` | Use the standard code, for example `it`, `nl`, `pt-BR`, `zh-Hant`. |
| `Duplicate mod id` warning, one mod ignored | Two mods share an `Id` | Local copy wins over a Workshop subscription. Delete the old local test copy, or unsubscribe. |
| `0 translations loaded` | CSV not found or header wrong | With a `Files` list, paths are relative to the mod folder and may not leave it. Header must contain `Key` and `Translation`, spelled like that. |
| `File is missing required columns` | Header row missing or wrong delimiter | The file must be comma separated. Spreadsheet exports in some locales use semicolons, choose comma on export. |
| Many `Unknown key` warnings | Keys edited, or a key from another game version | Copy keys from the template exactly. Keys are case sensitive. |
| Text shows as garbage characters | File saved in the wrong encoding | Save as UTF-8. |
| A text keeps the English original | Row has an empty translation, or the key is not in the string tables | Empty translations are skipped on purpose. A few interface texts are not moddable yet. |
| Translation shows but a placeholder is missing or literal | Placeholder altered | Restore the token from `SourceString`, including braces and `##` markers. |
| Subscribed, but mod does not load | Item not downloaded yet, or Steam not running | The game requests the download and reloads once it arrives. Starting the game without the Steam client disables Workshop mods, local mods still load. |
| Changes in the CSV not visible | Mods load at start | Restart the game. Workshop updates reload automatically. |

If you need help, you can reach us on the [Steam forums](https://steamcommunity.com/app/2977260/discussions/) and on our [Discord](https://discord.com/invite/theguild).
