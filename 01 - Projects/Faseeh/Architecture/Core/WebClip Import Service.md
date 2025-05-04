---
Where ?: Main Process
---
# WebClip Import Service Overview

> [!Note] Description
> Handles the process of importing content triggered externally via the `faseeh://` custom protocol URI. It manages the OS-level integration (protocol handling, single instance locking), parses the incoming request, optionally fetches initial web content, and then reliably **transmits the raw import data payload to the Renderer process via IPC** for subsequent processing.

## Key Notes

> [!Summary] Key Responsibilities
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Protocol Registration & Handling:</span> Registers Faseeh as the handler for `faseeh://` and manages application launch/focus events triggered by these URIs, ensuring only a single instance handles the request.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">URI Parsing & Validation:</span> Decodes and validates the incoming URI (e.g., `faseeh://import?type=url&value=...&title=...`) to extract import parameters.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Initial Data Fetching (URLs):</span> Optionally performs the initial network request within the Main process to fetch root HTML content for `type=url` imports, keeping network activity separate.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">IPC Data Handoff (Event Emission to Renderer):</span> Packages the extracted parameters and any fetched content into a standardized raw data payload (e.g., `{ type: 'url', value: 'http://...', title: '...', fetchedContent?: '<html>...' }`). Sends this payload to the active Renderer process via a specific IPC channel (e.g., `'webclip-import-data-received'`), effectively acting as an event signalling new data is ready.

> [!warning] **Workflow (URI Import):**
> 1.  **OS Trigger:** OS detects `faseeh://` URI invocation and launches/focuses the Faseeh application instance, passing the URI.
> ---
> 2.  **Capture (Main):** `WebClip Import Service` (Main Process) captures the URI via `app` lifecycle events (`open-url` or `second-instance`).
> ---
> 3.  **Parse (Main):** Service parses the URI, decodes parameters (type, value, title, etc.).
> ---
> 4.  **Fetch (Main - Optional):** If `type` indicates a URL, the service *may* fetch the initial HTML content.
> ---
> 5.  **Package (Main):** Service creates a raw data payload object containing the parsed information and fetched content (if any).
> ---
> 6.  **Transmit (Main -> Renderer via IPC):** Service sends the payload to the active Renderer window using `mainWindow.webContents.send('webclip-import-data-received', rawDataPayload)`.
> ---
> 7.  ***(Processing continues in Renderer - Triggered by the IPC event, received and handled by the [[Content Adapter Registry]])***

> [!Tip] Rationale:
> Keeping this service in the Main process ensures reliable handling of OS-level protocol activation and keeps potentially blocking network requests (initial fetch) off the Renderer thread. It acts solely as a receiver and forwarder for external triggers.