---
Where ?: Renderer Process
Type: Core Service
---

# OCR Registry Service Overview

> [!Note] Description
> Manages the registration, lifecycle, and discovery of all available Optical Character Recognition (OCR) engine implementations within Faseeh. It acts as a central factory and registry, instantiating engine classes (derived from `BaseOCREngine`) provided by the core application or plugins, managing their initialization (`initialize()`) and shutdown (`shutdown()`), and allowing other services (primarily Content Adapters) to find and retrieve ready-to-use engine instances based on required capabilities.

> [!Danger] Important
> OCR is a resource-intensive process. for now, we will keep it in the renderer process for the sake of simplicity. However, in the future, we should consider a way to offload this to a dedicated utitlity service or worker process to avoid blocking the main UI thread.
## Key Notes

> [!Summary]+ Key Responsibilities:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Engine Registration (`register`):</span> Accepts `BaseOCREngine` subclass constructors and their associated `OCREngineInfo` from core/plugins, storing this registration information.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Engine Lifecycle Management:</span>
>     - **Instantiation & Initialization:** Lazily creates instances of registered engine classes (`new EngineClass(info)`) when first requested. Calls the instance's `async initialize()` method and caches the initialized instance.
>     - **Shutdown & Unregistration (`unregister`):** Retrieves the cached engine instance associated with an ID, calls its `async shutdown()` method, and then removes the instance and registration details.
>     - **Global Teardown (`shutdownAllEngines`):** Provides a method to shut down all currently active engines, typically called during application exit.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Engine Discovery & Provision (`getEngineById`, `findBestEngine`):</span> Provides methods for querying registered engines based on ID or criteria (languages, capabilities). Returns **initialized** `BaseOCREngine` instances ready for use.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">API Exposure (`via FaseehApp`):</span> Exposes its public methods (`register`, `unregister`, `getEngineById`, `findBestEngine`) via the `app.ocrEngines` facade for use by plugins and potentially other core services.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Core Engine Management:</span> Registers and manages the lifecycle of any default OCR engine(s) provided by Faseeh itself.

> [!Example]- Interaction:
> 1.  **Registration:** Core app or Plugin calls `app.ocrEngines.register(MyOCREngineClass, engineInfo)`. Registry stores the class and info.
> ---
> 2.  **Request:** [[Content Adapter]] calls `app.ocrEngines.findBestEngine({ languages: ['jpn'] })`.
> ---
> 3.  **Initialization (by Registry):** Registry finds `MyOCREngineClass`, creates `instance = new MyOCREngineClass(info)`, calls `await instance.initialize()`, caches `instance`.
> ---
> 4.  **Provision:** Registry returns the `instance` to the [[Content Adapter]].
> ---
> 5.  **Usage (by Adapter):** [[Content Adapter]] calls `await instance.performOCR(...)`.
> ---
> 6.  **Unregistration:** Plugin calls `app.ocrEngines.unregister('my-ocr-id')`.
> ---
> 7.  **Shutdown (by Registry):** Registry retrieves cached `instance`, calls `await instance.shutdown()`, removes instance/registration.

## Types Definitions

> [!Summary]- Search Criteria: `OCREngineFindCriteria`
> Defines the criteria for finding a suitable OCR engine. 
> ```ts
> export interface OCREngineFindCriteria {
>   /** List of desired language codes (ISO 639) the engine should support. */
>   languages: string[];
>   /** Optional list of capability tags the engine should possess 
>   (e.g., 'handwriting', 'comic', 'vertical-text'). */
>   capabilities?: string[];
>   /** Optional: Hint for preferred engine ID if multiple match criteria. */
>   preferredEngineId?: string;
> }
> ```

> [!summary] Main Interface : `IOCREngineRegistry`
> ```ts
> export interface IOCREngineRegistry {
> 
>   register(
> 	  engineInfo: OCREngineInfo, 
> 	  engineClass: typeof BaseOCREngine
>   ): void;
> 
>   unregister(id: string): Promise`<void>`;
> 
>   getEngineById(id: string): 
> 	  Promise<BaseOCREngine | null>;
> 
>   findBestEngine(criteria: OCREngineFindCriteria): 
> 	  Promise<BaseOCREngine | null>;
> 
>   listRegisteredEngines(): OCREngineInfo[];
> 
>   shutdownAllEngines(): Promise`<void>`;
> }
> ```