---
Where ?: Renderer Process
Type: Abstract Base Class
---

# OCR Engine Overview

> [!Note] Description
> The foundational abstract class that all specific OCR engine implementations within Faseeh must inherit from. It defines the standard contract for performing OCR operations and provides optional lifecycle hooks (`initialize`, `shutdown`) for managing engine-specific resources (like loading models or managing worker processes). Instances of subclasses are managed and provided by the [[OCR Registry Service]].

## Key Notes

> [!Summary]+ Key Responsibilities & Features:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">OCR Contract (`performOCR` abstract method):</span> Defines the mandatory asynchronous `performOCR(imageData, languages, options?)` method that all subclasses must implement. This method takes image data and language hints and returns a standardized `OCRExtractionResult` containing the recognized text and bounding box information.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Optional Lifecycle Hooks (`initialize`, `shutdown`):</span> Provides optional `async initialize()` and `async shutdown()` methods. Subclasses can override these to perform setup tasks (e.g., loading models, starting workers) when the engine instance is first created/activated by the [[OCR Registry Service]], and teardown tasks (releasing memory/processes) when the engine is unregistered or the application closes.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Engine Metadata Access (`info` property):</span> Stores the `OCREngineInfo` metadata (ID, name, supported languages, capabilities) provided during registration, making it accessible within the engine instance via `this.info`.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Encapsulation:</span> Encapsulates the specific logic and potential state of a particular OCR engine implementation (e.g., the Tesseract.js worker instance, custom model paths).

> [!Example]- Interaction & Usage:
> 1.  **Plugin Definition:** A developer creates `MyCustomOCREngine extends BaseOCREngine`, implementing `performOCR` and optionally `initialize`/`shutdown`.
> ---
> 2.  **Registration:** The plugin registers its *class* (`MyCustomOCREngine`) and metadata (`OCREngineInfo`) with the [[OCR Registry Service]] via `app.ocrEngines.register(...)`.
> ---
> 3.  **Instantiation & Initialization:** The [[OCR Registry Service]], when needed, creates an instance `engine = new MyCustomOCREngine(info)` and calls `await engine.initialize()`.
> ---
> 4.  **Usage:** A [[Content Adapter]] retrieves the initialized `engine` instance from the Registry and calls `await engine.performOCR(imageData, ...)`.
> ---
> 5.  **Shutdown:** When the plugin unregisters the engine or the app closes, the [[OCR Registry Service]] calls `await engine.shutdown()`.
## Types Definitions

> [!Summary]- Input : `OCRInputData`
> The content that should be processed by the OCR engine can be a binary buffer, an `ImageData` object, or a string representing a file path or URL.
> ```ts
> // Input data for OCR
> export type OCRInputData = Buffer | ImageData | string; 
> // Image buffer, ImageData, or potentially image file path/URL
> ```

> [!Summary]- Output : `OCRExtractionResult`
> The result of the OCR process, typically an object containing recognized text, bounding boxes, and other metadata.
> ```ts
> // Result structure from an OCR engine (example)
> export interface OCRExtractionResult {
>   lines: {
>     text: string;
>     boundingBox: BoundingBox;
>     words: { 
> 	    text: string; 
> 	    boundingBox: BoundingBox; 
> 	    confidence?: number; 
> 	    language?: string; 
> 	}[];
>     confidence?: number;
>   }[];
>   // Optional overall document properties if detectable
>   detectedLanguage?: string;
>   orientation?: number;
> }
> ```
> > [!Summary]- Bounding Box
> > This interface defines the bounding box of a recognized text region. It includes coordinates, dimensions, and the unit of measurement. useful to make it the region interactive.
> > ```ts
> > export interface BoundingBox { 
> > 	x: number; 
> > 	y: number; 
> > 	width: number; 
> > 	height: number; 
> > 	unit: 'px' | '%' | 'relative'; 
> > }
> > ```

> [!summary]- OCR Engine Metadata : `OCREngineInfo`
> Metadata about OCR engine's capabilities. Based on these information, the registry can select the best engine for a given task.
> ```ts
> export interface OCREngineInfo {
>   id: string;
>   name: string;
>   supportedLanguages: string[];
>   capabilities?: string[];
>   description?: string;
> }

> [!summary] Main Interface : `BaseOCREngine`
> ```ts
> export abstract class BaseOCREngine {
>     // Readonly info provided during registration/instantiation
>     public readonly info: OCREngineInfo;
> 
>     constructor(info: OCREngineInfo) {
>         this.info = info;
>     }
> 
>     /**
>      * Performs the core OCR operation. MUST be implemented by subclasses.
>      * @param imageData The image data to process.
>      * @param languages Hints for languages to detect.
>      * @param options Engine-specific options.
>      * @returns A promise resolving to the structured OCR result.
>      */
>     abstract performOCR(
>         imageData: OCRInputData,
>         languages: string[],
>         options?: Record<string, any>
>     ): Promise<OCRExtractionResult>;
> 
> 	// TODO: defualt or abstract ?
>     async initialize(): Promise<void> {
>         console.log(`Engine ${this.info.id}: Default initialize.`);
>     }
> 
>     async shutdown(): Promise<void> {
>         console.log(`Engine ${this.info.id}: Default shutdown.`);
>     }
> }
> ```