---
Where ?: Renderer Process
Type: Abstract Base Class
---

# Subtitle Engine Overview

> [!Note] Description
> Defines the standard contract that all subtitle generation engine implementations must adhere to within Faseeh. Its primary purpose is to ensure a consistent interface (`generateSubtitles` method) for requesting subtitle generation from various sources (local files, potentially URLs) and returning results in a standardized format (`SubtitleGenerationResult`). It allows Faseeh to treat local models, 3rd-party APIs, and the Faseeh Cloud service interchangeably.

## Key Notes

> [!Summary]+ Key Responsibilities & Features:
>
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Generation Contract (`generateSubtitles` abstract method):</span> Defines the mandatory asynchronous `generateSubtitles(source, languages, options?)` method. Implementations take audio/video source data, target language hints, and optional configuration, returning a promise resolving to the standardized `SubtitleGenerationResult`.
>
> ---
>
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Engine Metadata Access (`info` property):</span> Stores the `SubtitleEngineInfo` (ID, name, supported languages, model details, etc.) provided during registration, making it accessible within the engine instance (`this.info`) if implemented as a class.
>
> ---
>
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Implementation Flexibility:</span> Subclasses handle the specific details:
>   - Loading/running local models (e.g., Whisper.cpp, Vosk).
>   - Calling 3rd-party cloud APIs (e.g., OpenAI Whisper, Google Speech-to-Text) using user-provided API keys (retrieved securely).
>   - Calling the Faseeh Cloud backend API.
>
> ---
>
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Optional Lifecycle Hooks (`initialize`, `shutdown`):</span> Allows engines requiring setup (loading models, initializing API clients) or teardown (releasing resources) to implement these optional asynchronous methods, managed by the [[Subtitle Engine Registry]].
>
> ---
>
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Standardized Output (`SubtitleGenerationResult`):</span> Ensures all engines return subtitle data in a consistent format (e.g., an array of timed text segments), regardless of the underlying technology.

> [!Example]- Interaction & Usage:
>
> 1.  **Plugin Definition:** Developer creates `CloudSubtitleEngine extends BaseSubtitleEngine`, implementing `generateSubtitles` to call a specific cloud API using keys loaded via `this.app.storage.loadData()`. May implement `initialize` to create an API client instance.
>
> ---
>
> 2.  **Registration:** Plugin registers `CloudSubtitleEngine` class and its metadata (supported languages, info about the cloud service) with the [[Subtitle Engine Registry]] via `app.subtitleEngines.register(...)`.
>
> ---
>
> 3.  **Invocation:** [[Subtitle Engine Registry]] provides an initialized instance to a requesting service (e.g., [[Content Adapter]]).
>
> ---
>
> 4.  **Execution:** The requesting service calls `await engineInstance.generateSubtitles(audioSource, ['en'])`. The engine's method handles API key retrieval, network request, response parsing, and formatting into `SubtitleGenerationResult`.

## API Types

> [!Summary]- Input Source : `SubtitleSourceData`
>
> ```ts
> // Input data for subtitle generation
> export type SubtitleSourceData = Buffer | string;
> // Audio/Video file content as Buffer, or file path/URL string
> ```

> [!Summary]- Output : `SubtitleGenerationResult`
>
> ```ts
> // Standardized output format for generated subtitles
> export interface SubtitleSegment {
>   startTimeMs: number;
>   endTimeMs: number;
>   text: string;
>   confidence?: number; // Optional confidence score
> }
>
> export interface SubtitleGenerationResult {
>   language: string; // Detected or confirmed language (ISO 639 code)
>   segments: SubtitleSegment[];
>   engineInfo?: {
>     // Info about the engine/model used
>     id: string;
>     name: string;
>     model?: string; // Specific model identifier if applicable
>   };
> }
> ```

> [!Summary]- Engine Metadata : `SubtitleEngineInfo`
>
> ```ts
> // Metadata describing a Subtitle Engine's capabilities
> export interface SubtitleEngineInfo {
>   id: string; // Unique ID (e.g., "faseeh-cloud-stt", "plugin-local-whisper")
>   name: string; // Human-readable name (e.g., "Faseeh Cloud", "Local Whisper (Medium)")
>   supportedLanguages: string[]; // Array of ISO 639 codes it can transcribe
>   inputType: ("audio" | "video" | "url")[]; // What kind of source it accepts
>   description?: string; // Details about the engine/model/service
>   requiresApiKey?: boolean; // Does this engine need a user-provided key via plugin settings?
>   isCloudService?: boolean; // Is processing done via a network request?
> }
> ```

> [!Summary]- Main Interface / Class : `BaseSubtitleEngine`
>
> ```ts
> export abstract class BaseSubtitleEngine {
>   public readonly info: SubtitleEngineInfo;
>
>   constructor(info: SubtitleEngineInfo) {
>     this.info = info;
>   }
>
>   /**
>    * Generates subtitles for the given source. MUST be implemented by subclasses.
>    * @param source The audio/video data or path/URL.
>    * @param languages Language hints (target languages).
>    * @param options Engine-specific options (e.g., quality settings, task type).
>    * @returns A promise resolving to the structured subtitle result.
>    */
>   abstract generateSubtitles(
>     source: SubtitleSourceData,
>     languages: string[], // May only support one target lang depending on engine
>     options?: Record<string, any>
>   ): Promise\<SubtitleGenerationResult>;
>
>   // Optional lifecycle methods
>   async initialize(): Promise\<void> {}
>   async shutdown(): Promise\<void> {}
> }
> ```
