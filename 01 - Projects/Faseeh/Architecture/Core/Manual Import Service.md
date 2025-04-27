---
Where ?: Renderer Process
---
---
# Manual Import Service Overview

> [!Note] Description
> Handles import triggers initiated directly by the user *within* the Faseeh UI (drag-and-drop, paste, file selection). Reads the initial data and sends it to the [[Content Adapter Registry]]

## Key Notes

> [!Summary] Key Responsibilities 
> - UI Event Handling (drop, paste, file input).
> -   Data Acquisition (reading files, clipboard).
> -   **Data Handoff:** Packages the acquired raw data (File path/Buffer, text content, URL string) into a standardized payload. Sends this payload to [[Content Adapter Registry]] via a direct call or through an local event emitter *(`Self Note:` avoid using `ipcRenderer` for this)*.

> [!warning] **Workflow (Manual Import):**
> 1.  User performs action (drag-drop, paste, file open) in UI (Renderer).
> 2. Capture event and read data.
> 3. Send raw data payload via local 
> 4. *(Processing continues in Main - See [[Content Adapter Registry]])*