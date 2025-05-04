---
Where ?: Main Process
---
# Storage Service Overview

> [!Note] Description
> The central authority for all persistent data management within Faseeh, operating exclusively in the **Main process**. It handles both the metadata database (SQLite) and the storage of associated files (media, extracted text, subtitles, document assets, plugin data, config) within the `.faseeh` directory structure. Crucially, all interactions from the Renderer process (UI, Plugins, Core Services like [[Content Adapter Registry]] and [[Plugin Manager]]) occur **indirectly via Electron's IPC mechanism**, brokered securely by the [[Storage Api]]. It directly executes the requested database and file system operations based on these validated IPC requests.

## Key Notes

> [!Summary]+ Key Responsibilities:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Secure IPC Endpoint for Storage Operations:</span> **Listens for and handles specific IPC messages** (via `ipcMain.handle`) initiated by the Renderer through the [[Storage Api]]. This is the *sole* entry point for Renderer-driven data manipulation.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Database Management:</span> Manages the application's SQLite database lifecycle (connection, schema, migrations) and executes all CRUD (Create, Read, Update, Delete) operations on metadata entities (`MediaObject`, `LanguageUnit`, `Collection`, etc.) as requested via IPC.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">File System Management:</span> Performs all file I/O operations within the managed `.faseeh` directory structure (saving/reading/deleting media files, extracted text, document assets, subtitles, plugin data, config files) using Node.js `fs` within the Main process, as directed by IPC requests. Ensures logical file organization (e.g., using UUIDs).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Data Integrity & Consistency:</span> Enforces relationships between database records and manages associated file operations consistently (e.g., deleting files when a DB record is deleted). Aims to handle concurrent IPC requests reliably (using `async/await` and potentially database transaction mechanisms).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Path Management:</span> Defines and provides canonical paths for application data directories (user data, media, plugins, config, etc.), responding to specific path retrieval requests via IPC.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">External Change Synchronization (Optional):</span> Optionally monitors managed directories for external file system changes (using `chokidar`) and reconciles the database, potentially notifying the Renderer via IPC.

> [!Example]- Workflow (Handling Renderer Request via IPC - e.g., Save MediaObject):
> 1.  **Initiation (Renderer):** The [[Content Adapter Registry]] has processed data and needs to save it.
> ---
> 2.  **API Call (Renderer -> Preload):** Calls `faseehStorageAPI.saveMediaObject(processedData)`.
> ---
> 3.  **IPC Invoke (Preload -> Main):** The Preload script sends an IPC message (e.g., `'storage:saveMediaObject'`) with `processedData` to the Main process.
> ---
> 4.  **Reception & Execution (Main):** The `Storage Service`'s `ipcMain.handle('storage:saveMediaObject', ...)` listener receives the data. It performs the necessary database `INSERT` and potentially saves related files (`fs.writeFile`).
> ---
> 5.  **Response Preparation (Main):** Prepares the result (e.g., `{ success: true, id: newMediaObjectId }` or an error).
> ---
> 6.  **IPC Return (Main -> Preload -> Renderer):** The result is returned through the resolved `ipcMain.handle` promise, back to the `ipcRenderer.invoke` in the Preload script, which resolves the promise for the original caller ([[Content Adapter Registry]]).
> ---
> 7.  **Completion (Renderer):** The [[Content Adapter Registry]] receives the result and proceeds (e.g., saving related assets using subsequent IPC calls, emitting `vaultEvents.emit('media:saved', ...)`).

> [!Example]- Workflow (Optional - External File Change Sync):
> 8.  **Monitoring (Main):** `Storage Service` watches folders using `chokidar`.
> ---
> 9.  **Detection (Main):** External file change detected.
> ---
> 10.  **Reconciliation (Main):** `Storage Service` updates its database records.
> ---
> 11.  **(Optional) Notification (Main -> Renderer):** Sends an IPC message (`'storage-external-change'`) if UI needs updating.
> ---
> 12.  **(Optional) UI Update (Renderer):** Renderer components listen for the IPC message and refresh.

> [!Tip] Interaction Summary:
> - **Receives Requests From:** Primarily the Renderer process via the [[Storage Api]] (`ipcMain.handle`). Callers include [[Content Adapter Registry]], [[Plugin Manager]], UI components, and indirectly [[Base Plugin|Plugins]] (via `this.app.storage`).
> - **Calls:** Uses Node.js `fs` and database driver APIs directly within the Main process.
> - **May Call (Optional):** Sends one-way IPC notifications (`webContents.send`) to the Renderer for events like external changes.

> [!FAQ] Potential Directory Structure
> ```
> .faseeh/
> ├── metadata.sqlite
> ├── library/
> │   └── {media_object_uuid}/
> │       ├── source.[ext]
> │       ├── document.json
> │       ├── assets/
> │       │   └── {asset_id}.[ext]
> │       └── associated/
> │           └── {type}_{lang?}_{name?}.[ext]
> ├── models/
> │   └── {model_name_or_id}/
> ├── plugins/
> │   └── {plugin-id}/
> │       ├── main.js
> │       ├── manifest.json
> │       └── data.json
> └── config/
>     └── enabled-plugins.json
>     └── settings.json
> ```
