---

Type: File
---
# `enabled-plugins.json` Overview

> [!Note] Description
> A JSON file storing an array or map of the unique IDs (`manifest.id`) of all community plugins that the user has currently enabled in their Faseeh library.
## Key Notes

> [!Summary]+ Content:
> - Typically an array of strings `["plugin-id-1", "plugin-id-3", ...]` or a map `{ "plugin-id-1": true, ... }`.
> ---
> - **Managed By:** Read at startup and updated by the [[Plugin Manager]] whenever a user enables or disables a plugin via the UI. Uses the [[Storage Api]] to interact with the [[Storage Service]] for persistence.
> ---
> - **Purpose:** Allows the [[Plugin Manager]] to determine which installed plugins should actually be loaded and activated during the startup sequence.
