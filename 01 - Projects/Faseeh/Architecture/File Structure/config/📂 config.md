---

Type: Folder
---

# `config` Directory Overview

> [!Note] Description
> Contains core configuration files for the Faseeh application itself and its management of extensions like plugins. These are typically simple JSON files storing application-wide settings or state.

## Key Notes

> [!Summary]+ Content:
> - **[[⚙️ enabled-plugins.json]]:** Stores the list of plugin IDs that the user has enabled.
> - **[[⚙️settings.json]]:** Stores core Faseeh application settings (e.g., UI preferences, default behaviors).
> - *(May contain other core configuration files as needed)*.
> ---
> - **Managed By:** Files are read/written by relevant core services (e.g., [[Plugin Manager]] for `enabled-plugins.json`, a potential Settings Service for `settings.json`) via requests to the [[Storage Service]] using the [[Storage Api]].
> ---
> - **Scope:** Contains application-level configuration, distinct from individual plugin data or library content metadata.