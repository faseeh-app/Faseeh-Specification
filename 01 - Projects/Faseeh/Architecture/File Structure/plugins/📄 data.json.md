---
Type: File
---
# `data.json` (Plugin) Overview

> [!Note] Description
> An optional JSON file automatically created and managed for a plugin if it uses the `this.loadData()` or `this.saveData()` helper methods provided by the [[Base Plugin]] class. It serves as the plugin's dedicated persistent storage for its settings, state, or other JSON-serializable data.
## Key Notes

> [!Summary]+ Content:
> - Arbitrary JSON data defined and used by the specific plugin. Typically contains configuration settings or cached state.
> ---
> - **Managed By:** Read from and written to by the [[Storage Service]] (Main Process) **only** when requested by the specific plugin via the `loadData()` / `saveData()` helpers (which use the [[Storage Api|Preload Storage API]] internally).
> ---
> - **Optionality:** This file only exists if the plugin actually saves data using the provided helpers.
> ---
> - **Scope:** Each plugin gets its own separate `data.json`, ensuring data isolation between plugins.

