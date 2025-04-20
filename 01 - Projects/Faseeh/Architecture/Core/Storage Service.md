---
Where ?: Main Process
---
# Storage Service Overview

> [!Note] Description
> Handles all persistent data management for Faseeh, including metadata database (SQLite) and associated files (media, extracted text, subtitles, etc.). It provides a low-level interface for creating, reading, updating, and deleting data, and also defining and managing paths. The service is responsible for managing the `.faseeh` directory structure and ensuring that all data is stored correctly.
## Key Notes

> [!Summary] Key Responsibilities 
>- <span style="font-weight:bold; color:rgb(146, 208, 80)">Unified Data Persistence</span> 
>  Provide the interface and logic for managing both the metadata database and associated file system storage within the .faseeh directories. Handle requests for creating, reading, updating, and deleting data entities and their corresponding files.
>- <span style="font-weight:bold; color:rgb(146, 208, 80)">Concurrency</span> 
>  While the main UI and plugins run in the renderer, heavy I/O or parsing might be offloaded to worker threads or even a utility process later *(and of course this is in the main process)*. The Storage Service API needs to handle concurrent access if that occurs (e.g., using queues, transactions).
>  > ⚠️ Initially we can use simple `async`/`await` operations (Node.js handles this internally, but we should ensure that the service is not blocking the main thread).
>- <span style="font-weight:bold; color:rgb(146, 208, 80)">External Change Synchronization (Optional)</span> 
>  Optionally monitor managed directories (e.g., .faseeh/media/) for file system changes made outside the application (using libraries like chokidar) and reconcile the metadata database accordingly.
>- <span style="font-weight:bold; color:rgb(146, 208, 80)">Secure IPC Endpoint</span> 
>  Expose all storage operations exclusively through Electron's IPC mechanism for controlled and secure access by the Renderer process (UI, plugins via Preload bridge).
>- <span style="font-weight:bold; color:rgb(146, 208, 80)">Path Management</span> 
>  Define and provide canonical paths for application data directories (user data, media, plugins, etc.) accessible via specific IPC requests.

> [!Example]  **Workflow (Handling Renderer Request):**
> 1.  <span style="color:rgb(0, 176, 240)">**Request:**</span> The Renderer process (UI, Plugin, or Service) needs a storage action `(save data, get data)`.
> 2.  <span style="color:rgb(0, 176, 240)">**Secure Bridge:**</span> The request goes through the Preload script [[Storage Api]].
> 3.  <span style="color:rgb(0, 176, 240)">**Transmission:** </span>The request and necessary data are sent from the Renderer to the Main process.
> 4.  <span style="color:rgb(0, 176, 240)">**Reception:**</span> The `Storage Service` in the Main process receives the request.
> 5.  <span style="color:rgb(0, 176, 240)">**Execution:**</span> The `Storage Service` performs the requested database and/or filesystem operations `(saves the file, updates the database record)`.
> 6.  <span style="color:rgb(0, 176, 240)">**Response:**</span> The `Storage Service` prepares the outcome `(success status, requested data, error message)`.
> 7. <span style="color:rgb(0, 176, 240)"> **Return:**</span> The outcome is sent back from the Main process to the Renderer process.
> 8.  <span style="color:rgb(0, 176, 240)">**Completion:**</span> The originating component in the Renderer receives the outcome and acts accordingly.

> [!Example]  **Workflow (Optional - External File Change Sync):**
> 1.  <span style="color:rgb(0, 176, 240)">**Monitoring:**</span> The `Storage Service` (Main process) watches designated application folders for changes made outside the app.
> 2.  <span style="color:rgb(0, 176, 240)">**Detection:**</span> The `Storage Service` detects changes (additions, deletions, modifications).
> 3. <span style="color:rgb(0, 176, 240)">**Reconciliation:**</span> The `Storage Service` reconciles the metadata database with the detected changes.
> 4. <span style="color:rgb(0, 176, 240)">**Notification:**</span> The `Storage Service` may notify the Renderer process of changes via IPC if necessary.
> 5. <span style="color:rgb(0, 176, 240)">**UI Update:**</span> The Renderer process updates the UI based on the reconciled data.

> [!FAQ] Potential Directory Structure
> ```
faseeh/
>     ├── database/
>     │   └── metadata.sqlite  # Or similar DB file
>     ├── media/             # Imported media files (videos, audio, PDFs)
>     │   └── {uuid}.[ext]
>     ├── text_content/      # Extracted text from articles, books etc.
>     │   └── {media_object_uuid}.txt
>     ├── subtitles/         # Generated or imported subtitles
>     │   └── {media_object_uuid}.json # or .srt
>     ├── models/            # Locally downloaded AI models
>     ├── plugins/           # Installed community plugins
>     │   └── plugin-id/
>     │       ├── main.js
>     │       ├── manifest.json
>     │       └── data.json  # Plugin-specific saved data
>     └── config/            # Core app configuration files
>         └── enabled-plugins.json
>         └── settings.json
> ```
