---
Type: Folder
---
# `plugins` Directory Overview

> [!Note] Description
> The root directory where all installed community (third-party) plugins reside. Each installed plugin occupies its own subdirectory within this folder.

## Key Notes

> [!Summary]+ Content:
> - **Plugin Subfolders (`{plugin-id}/`):** Contains subdirectories, each named with the unique ID of an installed plugin.
> ---
> - **Managed By:**
>     - **Discovery:** Scanned by the [[Plugin Manager]] (Renderer Process) at startup to find installed plugins. Path obtained via [[Storage Api]].
>     - **Modification:** Folders/files are typically added/removed/updated by a plugin installation/update mechanism (which could be part of the Plugin Manager or a separate process, likely needing Storage API interaction).
> ---
> - **Access:** Code within (`main.js`) is loaded directly by the [[Plugin Manager]] using `require()`. Manifests (`manifest.json`) are read by the Plugin Manager. Plugin data (`data.json`) is read/written via the [[Storage Api]] initiated by the plugin itself.

---
# `plugins/{plugin-id}` Directory Overview

> [!Note] Description
> Represents the dedicated container folder for a **single installed community plugin**. The folder name `{plugin-id}` is the unique identifier defined in the plugin's `manifest.json`.
## Key Notes

> [!Summary]+ Content:
> - **[[📄 manifest.json]] (Required):** Metadata about the plugin.
> - **[[📄 main.js]] (Required):** The plugin's compiled JavaScript entry point.
> - **[[📄 data.json]] (Optional):** Plugin-specific persistent data.
> - **Other Assets (Optional):** May contain CSS files (`styles.css`), other JavaScript modules, images, or resources required by the plugin.
> ---
> - **Managed By:** Created/updated by the plugin installation/update process. Read by the [[Plugin Manager]] (for manifest/main.js) and [[Storage Service]] (for data.json via plugin helpers).