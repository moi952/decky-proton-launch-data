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
    }
  ],
  "locales": {
    "en-US": {
      "categories": {
        "performance": "Performance"
      },
      "variables": {
        "dxvk_async": "DXVK Async",
        "enable_prefix": "Enable",      // required in every locale
        "disable_prefix": "Disable"     // required in every locale
      }
    },
    "fr-FR": { "...": "..." }
  }
}
```

## Notes

- `enable_prefix` and `disable_prefix` are required in every locale
- `favorites` and `custom` category names are managed by the plugin itself, not here
- Supported locales: `en-US` `fr-FR` `de-DE` `es-ES` `it-IT` `ja-JP` `ko-KR` `nl-NL` `pl-PL` `pt-BR` `pt-PT` `ru-RU` `tr-TR` `uk-UA` `zh-CN`
