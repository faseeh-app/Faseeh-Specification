---
Where ?: Main Process
---
---
# WebClip Import Service Overview

> [!Note] Description
> Handles the process of importing content triggered externally via the `faseeh://` custom protocol URI. It handles the OS-level integration, initiates content fetching/adaptation, and triggers storage, acting as the primary coordinator for web-clipped or URI-based imports within the **Main process**.
## Key Notes

> [!Summary] Key Responsibilities 
> - <span style="font-weight:bold; color:rgb(255, 192, 0)">Protocol Registration</span> 
>   Registers Faseeh as the default handler for the `faseeh://` protocol using `app.setAsDefaultProtocolClient('faseeh')`.
> - <span style="font-weight:bold; color:rgb(255, 192, 0)">URI Event Handling</span>
>   Listens for application launch/focus events triggered by `faseeh://` URIs.
> - <span style="font-weight:bold; color:rgb(255, 192, 0)">URI Parsing & Validation</span>
>   Parses the incoming URI (e.g., `faseeh://import?type=url&value=...&title=...`) and performs basic validation.
> - <span style="font-weight:bold; color:rgb(255, 192, 0)">Initial Data Fetching (Optional - URLs)</span>
>   May optionally perform the initial network request to fetch HTML content for `type=url` imports directly within the Main process to keep network activity separate from the Renderer
> - <span style="font-weight:bold; color:rgb(255, 192, 0)">Event Emission</span>
>   Emits `webclip-import-data-received` event to the renderer process to trigger content adaptation.

> [!warning] **Workflow (URI Import):**
> 1.  OS detects `faseeh://` URI invocation.
> 2.  OS launches/focuses Faseeh, passing the URI.
> 3.  `Media Importing Service` (Main Process) captures the URI via `app` events.
> 4.  Service parses the URI and decodes parameters.
> 5.  (Optional) If `type=url`, service fetches initial HTML content.
> 6.  Service emits a `webclip-import-data-received` event to Renderer process.
> 7. *(Processing continues in Main - See [[Content Adapter Registry]])*