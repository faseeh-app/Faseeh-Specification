---
Where ?: Renderer Process
Type: Abstract Base Class
---

# Base Content Adapter Overview

> [!Note] Description
> An optional base class or interface defining the standard contract for Content Adapters in Faseeh. Its primary purpose is to ensure all adapters implement the core `adapt` method, which takes raw source data and transforms it into a structured `ContentAdapterResult` containing metadata, potentially a [[Content Adapter#^205d65|FaseehContentDocument]], extracted assets, and associated files.

## Key Notes

> [!Summary]+ Key Responsibilities & Features:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Adaptation Contract (`adapt` abstract method):</span> Defines the mandatory asynchronous `adapt(source, context)` method that adapter implementations must provide. This method contains the core logic for parsing the input, potentially calling other services (like OCR), and structuring the output.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Adapter Metadata Access (`info` property):</span> If implemented as a class, it stores the `ContentAdapterInfo` metadata provided during registration, making it accessible via `this.info`.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Stateless Nature (Typical):</span> Content adapters are often designed to be relatively stateless, focusing purely on the transformation logic within the `adapt` method. They don't usually require complex lifecycle methods like `initialize` or `shutdown`.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Context/API Access:</span> The `adapt` method receives a `context` object (passed by the `ContentAdapterRegistry`) which provides necessary access to other application parts, such as the OCR engine registry (`context.app.ocrEngines`) or potentially a limited storage API facade (`context.app.storage`) if needed during adaptation itself (though final saving is handled by the Registry).

> [!Example]- Interaction & Usage:
> 1.  **Plugin Definition:** `MyPdfAdapter extends BaseContentAdapter` (or defines an object matching `ContentAdapterRegistration` including an `adapt` function).
> ---
> 2.  **Registration:** Plugin registers its adapter *class* or *function* and metadata (`ContentAdapterInfo`) with the `ContentAdapterRegistry` via `app.contentAdapters.register(...)`.
> ---
> 3.  **Invocation:** `ContentAdapterRegistry` selects this adapter, potentially instantiates it if a class was registered, and calls `await adapterInstanceOrFunction.adapt(sourceFile, context)`.
> ---
> 4.  **Execution:** The adapter's `adapt` method runs, potentially using `context.app.ocrEngines` to get an OCR engine, process the file, and returns the `ContentAdapterResult`.

## Types Definitions
### I/O types

> [!Summary]- Input : `ContentAdapterSource`
> The content to be processed, it can be a `File` object, a binary `Buffer`, a URL string, or a pasted text string.
> ```ts
> /** Represents the source input for an adapter */
> export type ContentAdapterSource = File | Buffer | string;
> // File from input/drop, Buffer from file read, URL string, or pasted text string
> ```

> [!Summary]- Output : `ContentAdapterResult`
> The output from an adapter is a `ContentAdapterResult` object, which contains the processed information ready for storage and use within Faseeh. It includes:
> - **`mediaObjectData`**: (`Partial<MediaObject>`) Core metadata about the original source (like guessed type, name, language). Some initial metadata might be provided by the triggering service ([[Manual Import Service]] or [[WebClip Import Service]]), and the adapter can enrich or refine it based on the content analysis.
> ---
> - **`contentDocument`** (Optional): ([[Content Adapter#^205d65|FaseehContentDocument]]) A structured representation of the main content, used for complex formats like articles, books, or comics/manga where layout, embedded media, or annotations are important. It contains an ordered list of `ContentBlock`s (text, images, annotations, etc.) and metadata about embedded assets. This field is omitted if the source doesn't have structured content suitable for this format (e.g., a simple audio file or a video URL).
> ---
> - **`documentAssets`** (Optional):  A map containing binary assets (like images extracted from an article or PDF pages) that are referenced within the `contentDocument`. The key is an internal `assetId` used within the document's blocks, and the value contains the raw `Buffer` content and format, ready to be saved by the Storage Service.
> ---
> - **`associatedFiles`** (Optional): An array holding supplementary files or data streams that are related to the `MediaObject` but are not part of the primary `contentDocument` structure. Common examples include subtitle files (`.srt`, `.vtt`), chapter markers (`.json`), alternative audio tracks, or extracted metadata files.
> ---
> ```ts
> export interface ContentAdapterResult {
>   mediaObjectData: Partial`<MediaObject>`; 
>   
>   contentDocument?: FaseehContentDocument;
>
>   documentAssets?: {
> 	  // Key matches assetId used within contentDocument
>     `[assetId: string]`: { 
>       format: string;    // e.g., 'jpeg', 'png', 'mp4', 'webp'
>       content: Buffer;   // The raw file
>     }
>   };
>
>   associatedFiles?: {
> 	 // e.g., 'subtitle', 'chapters', 'audioTrack', 'metadata'
>       type: string;        
>       format?: string;  // e.g., 'srt', 'vtt', 'json', 'mp3', 'xml'
>       language?: string; // ISO 639 code if applicable
>       filename?: string;  // Optional original or suggested filename
>       content: string | Buffer; // The content itself
>   }[];
> }
> ```

---

### Adapter Definition Types

> [!Summary]- Adapter Logic : `ContentAdapterFunction`
> Defines the signature for the core function that performs the content adaptation. It asynchronously takes the source data and optional context (like API access or original path) and returns the processed `ContentAdapterResult`.
> ```ts
> /** Function signature for adapter logic */
> export type ContentAdapterFunction =
>     (source: ContentAdapterSource, context: { // Provide necessary context/API access
>         app: Pick<FaseehApp, 'ocrEngines' | 'storage'>; // Example: Limited API access
>         originalPath?: string;
>         sourceUrl?: string;
>     }) => Promise`<ContentAdapterResult>`;
> ```

> [!Summary]- Adapter Metadata : `ContentAdapterInfo`
> Describes the capabilities and identity of a Content Adapter. This information is used by the [[Content Adapter Registry]]]] to select the appropriate adapter for a given input source.
> ```ts
> /** Metadata describing a Content Adapter's capabilities */
> export interface ContentAdapterInfo {
>   id: string;                          // Unique ID (e.g., "faseeh-core-text", "plugin-youtube-adapter")
>   name: string;                        // Human-readable name
>   supportedMimeTypes?: string[];       // e.g., ["text/plain", "application/pdf", "image/jpeg"]
>   supportedExtensions?: string[];      // e.g., [".txt", ".md", ".pdf", ".epub"]
>   urlPatterns?: string[] | RegExp[];   // Patterns/regex to match handled URLs (e.g., "https://www.youtube.com/watch*")
>   canHandlePastedText?: boolean;       // If adapter should be considered for raw pasted text
>   priority?: number;                   // Higher number = higher priority for the same match type (default 0)
>   description?: string;                // Optional description for UI/debugging
> }
> ```

> [!Summary]- Adapter Registration : `ContentAdapterRegistration`
> The complete object used when registering an adapter with the [[Content Adapter Registry]]. It combines the metadata (`ContentAdapterInfo`) with the actual implementation logic (`adapt` function or potentially an adapter class constructor).
> ```ts
> /** The complete registration object provided by core or plugins */
> export interface ContentAdapterRegistration extends ContentAdapterInfo {
>   // Option 1: Simple function registration
>   adapt: ContentAdapterFunction;
>
>   // Option 2: Class-based registration (uncomment if using BaseContentAdapter)
>   // adapterClass: typeof BaseContentAdapter;
> }
> ```
> > *(Note: The definition here primarily shows function-based registration for simplicity, but class-based is also viable as discussed previously).*

---

### Structured Content Document Types

> [!Summary]- Document Container : `FaseehContentDocument`
> The root object representing structured content extracted by an adapter. It holds document-level metadata, references to related binary assets (images, etc.), and the main sequence of content blocks that define the structure and content.
> ```ts
> /** Represents structured, processed content ready for display/interaction */
> export interface FaseehContentDocument {
>   version: string; // Schema version
>   metadata: {
>     title?: string;
>     language?: string;
>     // ... other document-level metadata ...
>   };
>   assets?: { // Describes assets STORED SEPARATELY by Storage Service
>     [assetId: string]: {
>       format: string;
>       originalSrc?: string;
>       width?: number;
>       height?: number;
>       // Refers to stored asset path implicitly or explicitly via Storage Service lookup
>     }
>   };
>   contentBlocks: ContentBlock[]; // Ordered array of content blocks
> }
> ```

^205d65

> [!Summary]- Content Blocks : `ContentBlock` (Union Type)
> Represents the different types of elements that make up the structured content document. Each specific block type inherits from `BaseBlock`.
> ```ts
> /** Union type for all possible content blocks */
> export type ContentBlock = TextBlock | ImageBlock | VideoBlock | AudioBlock | AnnotatedImageBlock | ContainerBlock;
>
> /** Basic properties common to all blocks */
> export interface BaseBlock {
>   id: string; // Unique within the document
>   order: number; // Display/reading order
>   position?: BoundingBox; // Optional absolute positioning for layout-sensitive content
> }
> ```

> [!Summary]- Text Block : `TextBlock`
> Represents a segment of text content, potentially with styling information.
> ```ts
> /** Represents a block of text */
> export interface TextBlock extends BaseBlock {
>   type: 'text';
>   content: string;
>   style?: string; // e.g., 'paragraph', 'h1', 'li', 'caption'
>   language?: string; // Specific language if different from document default
> }
> ```

> [!Summary]- Image Block : `ImageBlock`
> Represents an image, either embedded (referenced via `assetId`) or external.
> ```ts
> /** Represents an image */
> export interface ImageBlock extends BaseBlock {
>   type: 'image';
>   assetId?: string;       // Refers to key in FaseehContentDocument.assets (for embedded/managed images)
>   externalSrc?: string;   // URL if image is hosted externally
>   alt?: string;           // Alt text
>   caption?: string;       // Associated caption text
> }
> ```

> [!Summary]- Video Block : `VideoBlock`
> Represents an embedded video, likely linking to an external source or a managed asset.
> ```ts
> /** Represents a video embed */
> export interface VideoBlock extends BaseBlock {
>     type: 'video';
>     assetId?: string;       // Refers to a managed video asset
>     externalSrc?: string;   // URL to video (e.g., YouTube, Vimeo, or direct file)
>     // Could include references to associated subtitle files managed elsewhere
> }
> ```

> [!Summary]- Audio Block : `AudioBlock`
> Represents embedded audio.
> ```ts
> /** Represents an audio embed */
> export interface AudioBlock extends BaseBlock {
>     type: 'audio';
>     assetId?: string;       // Refers to a managed audio asset
>     externalSrc?: string;   // URL to audio file
> }
> ```

> [!Summary]- Annotated Image Block : `AnnotatedImageBlock`
> Represents a base image (like a comic page or diagram) with multiple text regions overlaid, each having its own text, position, and type.
> ```ts
> /** Represents an image with annotated text regions (comics, diagrams) */
> export interface AnnotatedImageBlock extends BaseBlock {
>   type: 'annotatedImage';
>   baseImageAssetId: string; // Refers to key in FaseehContentDocument.assets
>   annotations: ImageAnnotation[]; // Array of text annotations on this image
> }
> ```

> [!Summary]- Image Annotation : `ImageAnnotation`
> Defines a single piece of text overlaid on an `AnnotatedImageBlock`, including its content, position, and semantic type (like dialogue or sound effect).
> ```ts
> /** Represents a single text annotation on an image */
> export interface ImageAnnotation {
>   id: string; // Unique within the parent block
>   text: string;
>   boundingBox: BoundingBox; // Position RELATIVE TO THE BASE IMAGE
>   type: 'dialogue' | 'sfx' | 'narration' | 'label' | 'caption' | 'title' | 'other'; // Semantic type
>   order: number; // Reading order of annotations on the image
>   language?: string;
> }
> ```

> [!Summary]- Container Block : `ContainerBlock`
> A structural block used to group other `ContentBlock`s, allowing for nested layouts (e.g., sections, panels, figures).
> ```ts
> /** Represents a structural container for other blocks */
> export interface ContainerBlock extends BaseBlock {
>     type: 'container';
>     style?: string; // e.g., 'section', 'panel', 'figure', 'div'
>     children: ContentBlock[]; // Nested blocks within this container
> }
> ```

> [!Summary]- Bounding Box : `BoundingBox`
> Defines a rectangular area, typically used for positioning annotations or blocks within a coordinate system (pixels, percentage, or relative).
> ```ts
> /** Represents a positional bounding box */
> export interface BoundingBox {
>   x: number;      // Top-left X or Left offset
>   y: number;      // Top-left Y or Top offset
>   width: number;  // Width of the box
>   height: number; // Height of the box
>   unit: 'px' | '%' | 'relative'; // Coordinate system unit
> }
> ```

---

### Other Referenced Types (Placeholders)

> [!Summary]- Media Object : `MediaObject` (Conceptual)
> Represents the core database entry for an imported piece of content, holding primary metadata. Referenced here, but defined fully elsewhere (likely related to the Storage Service schema).
> ```ts
> // Placeholder for MediaObject type definition (assumed defined elsewhere)
> export declare type MediaObject = {
>     id: string;
>     type: string; // video, audio, article, book, image, comic?
>     name: string;
>     language?: string;
>     sourceUri?: string; // Original file path or URL
>     storagePath?: string; // Path within .faseeh/media if applicable
>     contentDocumentPath?: string; // Path to the associated FaseehContentDocument JSON
>     createdAt: Date;
>     updatedAt: Date;
>     // ... other metadata fields ...
> };
> ```

> [!Summary]- Faseeh Application API : [[FaseehApp]] (Conceptual)
> Represents the API object passed to adapters or plugins, providing access to necessary core functionalities like OCR engines or storage facades. Referenced here, but defined fully elsewhere.
> ```ts
> // Placeholder for FaseehApp type definition (assumed defined elsewhere)
> export declare type FaseehApp = {
>     ocrEngines: /* IOCREngineRegistry facade */ any;
>     storage: /* IStorageAPI facade */ any;
>     // ... other API modules ...
> };
> ```