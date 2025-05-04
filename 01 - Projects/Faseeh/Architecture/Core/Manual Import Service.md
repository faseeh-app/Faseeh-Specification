---
Where ?: Renderer Process
---
# Manual Import Service Overview

> [!Note] Description
> Handles import triggers initiated directly by the user *within* the Faseeh UI (e.g., drag-and-drop files, pasting text/URLs, using File->Open). It acquires the raw data from these UI interactions and passes it directly (within the Renderer process) to the [[Content Adapter Registry]] for processing.

## Key Notes

> [!Summary] Key Responsibilities
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">UI Event Handling:</span> Listens for user interactions within the UI that signify an import action (e.g., `drop` events on designated areas, `paste` events in specific inputs, responses from `<input type="file">`).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Data Acquisition:</span> Reads the raw data associated with the UI event (e.g., accessing `DataTransfer.files`, reading `navigator.clipboard`, getting `File` objects from inputs). This may involve asynchronous file reading (`FileReader` or Node.js `fs` if needed for Buffers).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Local Data Handoff (Triggering Content Adapters):</span> Packages the acquired raw data (`File` object, `Buffer`, text string, URL string) into a standardized payload. Directly triggers the [[Content Adapter Registry]] (also in the Renderer) to begin processing this data, typically by calling its `processSource` method (e.g., `this.contentAdapterRegistry.processSource(rawDataPayload)`). *(Alternatively, could use a strictly local Renderer-only event bus if preferred for decoupling within the Renderer)*.

> [!warning] **Workflow (Manual Import):**
> 1.  **User Action (Renderer):** User performs an import action (drag-drop, paste, file open) within the application's UI.
> 2.  **Capture & Read (Renderer):** `Manual Import Service` (Renderer) captures the relevant UI event and asynchronously reads the associated raw data (e.g., file content into a `Buffer` or `File` object, clipboard text/URL).
> 3.  **Package (Renderer):** Service creates a raw data payload object containing the acquired data and any relevant context (e.g., original filename).
> 4.  **Trigger Adaptation (Renderer -> Renderer):** Service directly calls the `processSource` method on the [[Content Adapter Registry]] instance, passing the raw data payload. *(Or emits to a local Renderer event bus listened to by the Registry).*
> 5.  ***(Processing continues in [[Content Adapter Registry]])***

> [!Tip] Rationale:
> This service lives in the Renderer as it directly interacts with UI events and DOM APIs (clipboard, file inputs, drag/drop) which are only available there. It acts as the entry point for user-initiated imports within the application window.