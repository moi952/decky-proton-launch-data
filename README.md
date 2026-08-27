# decky-proton-launch-data

Remote data repository for [decky-proton-launch](https://github.com/moi952/decky-proton-launch).

Updating this file takes effect in the plugin without releasing a new version.

## Single file

```
data.json
```

**URL:**
```
https://raw.githubusercontent.com/moi952/decky-proton-launch-data/main/data.json
```

No authentication required.

## How it works

On every plugin launch:
1. The plugin loads its local cache (`variables_cache.json` in Decky's settings dir)
2. Fetches this file in the background
3. If the response is valid and non-empty → replaces the cache and updates the UI
4. If the fetch fails or returns empty data → keeps the existing cache intact

First install: no cache exists → UI shows nothing until the first fetch completes.

## Structure

```jsonc
{
  "variables": [
    {
      "category": "performance",        // i18n key → categories namespace
      "variables": [
        {
          "title": "dxvk_async",        // i18n key → variables namespace
          "env": "DXVK_ASYNC",          // environment variable name
          "type": "bool",               // "bool" | "enum"
          "value": "1"                  // default value
        },
        {
          "title": "radv_perftest",
          "env": "RADV_PERFTEST",
          "type": "enum",
          "multiSelect": false,         // optional, "enum" only — false: exclusive pick, true: several values combinable (comma-joined). See Notes.
          "defaultValue": "aco",
          "values": [
            { "title": "radv_perftest_aco", "value": "aco" },
            { "title": "radv_perftest_gpl", "value": "gpl" }
          ]
        },
        {
          "title": "lsfg",
          "env": "__LSFG",
          "type": "bool",
          "value": "1",
          "simple": true                // on/off only, no Enable/Disable label
        }
      ]
    },
    {
      "category": "nvidia_commands",
      "variables": [ /* ... */ ],
      "subCategory": {                  // optional — a related group of variables shown under its own heading
        "title": "nvidia_commands_driver",       // i18n key → categories namespace
        "description": "nvidia_commands_driver_description", // optional, i18n key → descriptions namespace
        "variables": [
          {
            "title": "ngx_updater",
            "env": "PROTON_ENABLE_NGX_UPDATER",
            "type": "bool",
            "value": "1"
          },
          {
            "title": "dxvk_sr_override",
            "env": "DXVK_NVAPI_DRS_NGX_DLSS_SR_OVERRIDE",
            "type": "enum",             // OFF/ON/DEFAULT per dxvk-nvapi, not a plain bool — see Notes
            "multiSelect": false,
            "defaultValue": "default",
            "values": [
              { "title": "disable_prefix", "value": "off" },
              { "title": "enable_prefix", "value": "on" },
              { "title": "preset_default", "value": "default" }
            ],
            "subGroup": [              // optional — variables that only make sense once this one is enabled. Nest under the toggle it actually depends on, not an unrelated one.
              {
                "title": "dxvk_sr_preset",
                "env": "DXVK_NVAPI_DRS_NGX_DLSS_SR_OVERRIDE_RENDER_PRESET_SELECTION",
                "type": "enum",
                "multiSelect": false,
                "defaultValue": "default",
                "values": [
                  { "title": "preset_off", "value": "off" },
                  // "title" can reuse one templated i18n key across many values via titleParams
                  { "title": "preset_cnn", "titleParams": { "letter": "A" }, "value": "render_preset_a" }
                ]
              }
            ]
          }
        ]
      }
    }
  ],
  "conflictGroups": [                 // optional — top-level, sibling of "variables"/"locales"
    // "trigger" conflicts with each env in "conflicts" — a star, not a
    // clique: the entries in "conflicts" are NOT considered in conflict
    // with each other, only each one individually with "trigger".
    { "trigger": "PROTON_ENABLE_NGX_UPDATER", "conflicts": ["PROTON_DLSS4_UPGRADE"] },
    { "trigger": "PROTON_USE_WINED3D", "conflicts": ["DXVK_ASYNC", "PROTON_DLSS4_UPGRADE"] }
  ],
  "locales": {
    "en-US": {
      "categories": {
        "performance": "Performance"
      },
      "descriptions": {
        "nvidia_commands_driver_description": "Official NVIDIA driver variables..."
      },
      "variables": {
        "dxvk_async": "DXVK Async",
        "preset_cnn": "Preset {{letter}} (CNN)",   // {{letter}} filled in from the value's titleParams
        "enable_prefix": "Enable",      // required in every locale
        "disable_prefix": "Disable",    // required in every locale
        "conflict_warning": "May conflict with {{names}} — consider disabling it" // required if conflictGroups is used
      }
    },
    "fr-FR": { "...": "..." }
  }
}
```

## Notes

- `enable_prefix` and `disable_prefix` are required in every locale
- `favorites` and `custom` category names are managed by the plugin itself, not here
- `multiSelect` (`"type": "enum"` only, optional): `false` = exclusive single pick, `true` = several values selectable and combined (comma-joined) into the env var. Omit to keep the current default behavior.
- `titleParams` (optional, on an enum `values[]` entry): interpolation values passed alongside `title` so several options can share one templated i18n string (e.g. `"Preset {{letter}} (CNN)"`) instead of one key per option.
- `subCategory` (optional, on a top-level category block): a secondary, related group of variables rendered under its own heading, indented under its parent category. Toggling it off is a separate visibility switch in Settings, keyed by its own `title`.
- `subGroup` (optional, on a variable): nested variables that only make sense once the parent variable is enabled — nest under whichever toggle they actually depend on (e.g. a preset picker under its own SR/RR/FG override toggle, not an unrelated one). Turning the parent off also force-deactivates the whole subGroup, both in the UI and in what actually gets exported.
- A variable with a non-empty `subGroup` is always rendered "simple" (title only, no Enable/Disable label or 0/1 picker) regardless of `simple` — being on just means "take its value", nothing else to pick for that row itself.
- `conflictGroups` (optional, top-level): each entry is `{ trigger, conflicts }` — `trigger` conflicts with every env listed in `conflicts`, individually. It's a star, not a clique: the envs inside `conflicts` are NOT considered in conflict with each other, only each one with `trigger`. Use this for "A breaks B, C and D" (one env whose effect breaks several unrelated others) as well as plain pairs (a 2-element case behaves the same either way). Whenever `trigger` and one or more of its `conflicts` are active at once, every active one among them shows a red warning naming the other(s) — works regardless of which was turned on first. One generic `conflict_warning` key (with a `{{names}}` placeholder) covers every rule; each variable's own existing `title` translation is reused to fill it in, so adding a conflict never needs a new per-pair i18n key.
- `subCategory`, `subGroup`, `titleParams` and `multiSelect` are supported starting from the plugin version that shipped this doc update.
- Supported locales: `en-US` `fr-FR` `de-DE` `es-ES` `it-IT` `ja-JP` `ko-KR` `nl-NL` `pl-PL` `pt-BR` `pt-PT` `ru-RU` `tr-TR` `uk-UA` `zh-CN`
