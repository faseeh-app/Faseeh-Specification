---
Where ?: Renderer Process
Type: Core Service
---

#  Content Adapter Registry Overview

> [!Note] Description
> Manages all available strategies (Content Adapters) for interpreting and processing various types of raw content (files, URLs, pasted text) imported into Faseeh. It maintains a registry of adapters (built-in and plugin-provided), selects the most appropriate one for a given input source, invokes its processing logic (`adapt` method), and then orchestrates the multi-step saving of the results (metadata, structured content document, assets, associated files) via the [[Storage Api]].

## Key Notes

> [!Summary]+ Key Responsibilities:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Adapter Registry Management:</span> Maintains an internal collection of registered `ContentAdapterRegistration` objects (which include adapter metadata and the `adapt` function or adapter class constructor). Indexed for efficient lookup.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Registration Interface (`via FaseehApp`):</span> Exposes `app.contentAdapters.register` / `unregister` methods allowing core app and plugins to add/remove adapters, specifying their capabilities (MIME types, extensions, URL patterns, priority).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Core Fallback Adapters:</span> Includes built-in, low-priority adapters for basic types (e.g., plain text) ensuring baseline import functionality.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Best Adapter Selection:</span> Implements logic to find the most suitable registered adapter for a given input source based on type, name, URL pattern, and priority.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Import Processing Orchestration (`processSource` method):</span> The core public method that takes raw `ContentAdapterSource`. It performs the following steps:
>     1.  Selects the best adapter based on the source.
>     2.  Prepares necessary context/API access for the adapter (e.g., limited `FaseehApp` access, especially `app.ocrEngines`).
>     3.  Invokes the selected adapter's `adapt(source, context)` method asynchronously.
>     4.  Receives the `ContentAdapterResult`.
>     5.  **Coordinates Multi-Step Storage (via [[Storage Api]]):** Sequentially calls the necessary `faseehStorageAPI` methods (`saveMediaObject`, `saveDocumentAssets`, `saveContentDocument`, `saveAssociatedFiles`) using the data from the `ContentAdapterResult`, handling dependencies (like needing the `MediaObject` ID before saving related content).
>     6.  **Emits Events:** Dispatches success (`media:saved`) or failure (`media:import-failed`) events on the shared `vaultEvents` emitter.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Error Handling:</span> Manages errors during adapter selection, execution, or storage coordination, emitting appropriate events/notifications.

> [!Example]- Workflow (Processing Imported Data - e.g., URL):
> 1.  **Reception:** Registry receives a URL string via an internal method called by the IPC listener for `webclip-import-data-received`.
> ---
> 2.  **Selection:** Registry matches the URL pattern and finds a high-priority "Web Article Adapter" registered by a plugin.
> ---
> 3.  **Adaptation:** Registry calls `await webArticleAdapter.adapt(urlString, { app })`. The adapter fetches the page, uses readability/parsing libraries, potentially calls `app.ocrEngines` for images, and returns a `ContentAdapterResult` with `mediaObjectData`, a `contentDocument` (TextBlock, ImageBlock nodes), and `documentAssets` (downloaded images).
> ---
> 4.  **Storage (Coordination):** Registry uses the [[Storage Api]] to:
>     *   Save `mediaObjectData` -> Gets ID.
>     *   Save `documentAssets` (images) -> Gets stored paths/map.
>     *   Update asset paths in `contentDocument`, then save `contentDocument`.
>     *   Save any `associatedFiles`.
> ---
> 5.  **Notification:** Registry emits `vaultEvents.emit('media:saved', { id: newMediaObjectId, ...result.mediaObjectData })`.
> ---
> 6.  **UI Update:** UI components react.

> [!Tip] Integration:
> - Triggered by [[WebClip Import Service]] (via IPC) and [[Manual Import Service]] (locally).
> - Manages adapters registered via `FaseehApp.contentAdapters.register`.
> - **Crucially relies on the [[Storage Api]] for all persistence.**
> - May utilize other services like the `OCREngineRegistry` (indirectly, by providing access to adapters).
> - Emits events on shared [[Event Emitter Wrapper]] instances (`vaultEvents`).

> [!warning] Complexity Note:
> The multi-step saving process involving multiple asynchronous calls to the [[Storage Api]], needs careful implementation to handle potential failures at each step gracefully.

## API Types

### Registry Interaction Types

> [!Summary]- Adapter Selection Criteria : `ContentAdapterFindCriteria`
> Defines the information used by the [[Content Adapter Registry]]'s `findBestAdapter` method to identify the most suitable adapter for a given input source. The registry uses properties like MIME type, file extension, or URL patterns provided during adapter registration to find a match.
> ```ts
> /** Criteria used to find the best Content Adapter */
> export interface ContentAdapterFindCriteria {
>   /** The original input source (used for potential content sniffing or complex matching) */
>   source: ContentAdapterSource;
>   /** The MIME type of the source, if known (e.g., from File object or HTTP headers) */
>   mimeType?: string;
>   /** The file extension, if source is a file or has a relevant name */
>   fileExtension?: string;
>   /** The source URL, if the source is a URL string */
>   sourceUrl?: string;
>   /** Indicates if the source is raw pasted text */
>   isPastedText?: boolean;
> }
> ```

> [!Summary]- Registry Interface : `IContentAdapterRegistry` (Conceptual)
> Defines the public contract for the [[Content Adapter Registry]] service, outlining the methods available for registering, unregistering, and finding adapters. *(Note: we might implement the class directly without a separate interface, defining the interface is just to clarifies the contract).*
> ```typescript
> export interface IContentAdapterRegistry {
>
>   /**
>    * Registers a new Content Adapter implementation.
>    * @param registration - The complete registration object including metadata and the adapt logic.
>    */
>   register(registration: ContentAdapterRegistration): void;
>
>   /**
>    * Unregisters a Content Adapter by its unique ID.
>    * @param id - The unique ID of the adapter to unregister.
>    */
>   unregister(id: string): void;
>
>   /**
>    * Finds the most suitable registered Content Adapter based on the provided source info.
>    * Returns the full registration object, including the 'adapt' function.
>    * Returns null if no suitable adapter is found.
>    * @param criteria - Information about the source content to use for matching.
>    * @returns The best matching ContentAdapterRegistration, or null.
>    */
>   findBestAdapter(criteria: ContentAdapterFindCriteria): ContentAdapterRegistration | null;
>
>   /**
>    * Retrieves a specific Content Adapter registration by its ID.
>    * Returns null if no adapter with the given ID is registered.
>    * @param id - The unique ID of the desired adapter.
>    * @returns The ContentAdapterRegistration object, or null.
>    */
>   getAdapterById(id: string): ContentAdapterRegistration | null;
>
>   /**
>    * Retrieves metadata about all currently registered Content Adapters.
>    * @returns An array of ContentAdapterInfo objects.
>    */
>   listRegisteredAdapters(): ContentAdapterInfo[];
>
>   /**
>    * The primary method that takes raw source data, finds the best adapter,
>    * invokes it, and orchestrates the storage of the results via the Storage API.
>    * @param source - The raw input data (File, Buffer, URL string, text string).
>    * @param context - Optional context like original path or URL.
>    * @returns Promise resolving with success status and potentially the new MediaObject ID or error info.
>    */
>   processSource(
> 	  source: ContentAdapterSource, 
> 	  context?: { 
> 		  originalPath?: string; 
> 		  sourceUrl?: string 
> 	  }
>   ): Promise<{ 
> 	  success: boolean; 
> 	  mediaObjectId?: string; 
> 	  error?: string 
>   }>;
>
> }
> ```

