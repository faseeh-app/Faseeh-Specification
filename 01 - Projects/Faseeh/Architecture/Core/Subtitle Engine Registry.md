---
Where ?: Renderer Process
Type: Core Service
---

# Subtitle Engine Registry Service Overview

> [!Note] Description
> Manages the registration, discovery, and potentially lifecycle (if needed) of various subtitle generation engine implementations within Faseeh. It acts as a central registry where the core application and plugins can register different subtitle generation capabilities (local models, 3rd-party cloud services via API keys, or the Faseeh Cloud service). It allows other services (primarily Content Adapters) to find and utilize the most appropriate or user-selected engine for generating subtitles from audio or video sources.

## Key Notes

> [!Summary]+ Key Responsibilities:
>
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Engine Registration (`register`):</span> Provides an interface (likely via `app.subtitleEngines.register` in the [[FaseehApp]] API) for the core app and plugins to register implementations conforming to the [[Subtitle Engine|BaseSubtitleEngine]] contract. Registration includes the engine's implementation (class or function object) and descriptive metadata (`SubtitleEngineInfo`).
>
> ---
>
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Registry Management:</span> Maintains an internal collection of all registered subtitle engine implementations, indexed for efficient lookup based on ID or capabilities.
>
> ---
>
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Engine Discovery & Provision (`getEngineById`, `findBestEngine`)</span>Provides methods to query available engines. `findBestEngine` would take criteria like desired languages, source type (audio/video), and potentially user preference to select and return a ready-to-use engine instance (or function object). Handles potential lazy initialization if engines require setup.
>
> ---
>
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Lifecycle Management (Optional but Recommended)</span>Similar to OCR, if engines require setup/teardown (e.g., loading local models, managing API clients), the registry can manage calling optional `initialize()` / `shutdown()` methods on registered engine instances.
>
> ---
>
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">API Exposure (via [[FaseehApp]] )</span>Exposes its public methods (`register`, `unregister`, `getEngineById`, `findBestEngine`, `listRegisteredEngines`) via the `app.subtitleEngines` facade for use by plugins and potentially other core services.
>
> ---
>
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Core/Default Engine Handling</span>May register built-in implementations, including the facade for the Faseeh Cloud subtitle service.

> [!Example]- Interaction:
>
> 1.  **Registration:** A plugin registers its Whisper.cpp wrapper: `app.subtitleEngines.register(MyWhisperEngineClass, engineInfo)`.
>
> ---
>
> 2.  **Request:** A [[Content Adapter]] processing a video needs Japanese subtitles: `const engine = await app.subtitleEngines.findBestEngine({ source: videoFile, languages: ['ja'] });`.
>
> ---
>
> 3.  **Provision & Initialization (by Registry):** Registry finds `MyWhisperEngineClass`, potentially creates an instance (`new MyWhisperEngineClass(info)`), calls `await instance.initialize()` if defined, and returns the usable `instance`.
>
> ---
>
> 4.  **Usage (by Adapter):** Adapter calls `const subtitles = await engine.generateSubtitles(videoFile, ['ja'], { /* options */ });`.
>
> ---
>
> 5.  **Processing (by Engine):** The engine's `generateSubtitles` method runs (either locally or calling a 3rd-party/Faseeh Cloud API).
>
> ---
>
> 6.  **Result:** The engine returns the structured subtitle data (`SubtitleGenerationResult`).
>
> ---
>
> 7.  **Unregistration/Shutdown:** Managed by the registry when the plugin unloads or the app closes.