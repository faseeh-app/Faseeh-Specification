---

Type: File
---
# `settings.json` Overview

> [!Note] Description
> A JSON file storing core configuration settings for the Faseeh application itself. This includes user preferences related to UI appearance, default behaviors, core feature settings (if any), etc., but generally *not* data related to specific imported media or individual plugin settings (which are stored elsewhere).

## Key Notes

> [!Summary]+ Content:
> - A JSON object containing various key-value pairs for application settings (e.g., `"theme": "dark"`, `"defaultImportLanguage": "en"`, `"autoCheckUpdates": true`).
> ---
> - **Managed By:** Read at startup and potentially updated by a dedicated core Settings Service or directly by UI components responsible for settings management. Uses the [[Storage Api]] to interact with the [[Storage Service]] for persistence.
> ---
> - **Purpose:** Allows user preferences and application configurations to persist across sessions.

> [!Tip] Global Settings:
> Think of this as the place for application-wide preferences, separate from the library content or plugin-specific configurations.