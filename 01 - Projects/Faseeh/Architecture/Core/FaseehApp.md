---
Where ?: Renderer Process
Type: Core API Object / Facade
---
# FaseehApp API Object Overview

> [!Note] Description
> The [[FaseehApp]] object (typically accessed as `this.app` within a plugin instance) is the **primary API** provided to all loaded Faseeh plugins. It serves as a controlled and stable **facade**, abstracting the application's internal services and architecture. It provides plugins with the necessary methods and properties to interact with core functionalities (like storage, UI, plugin management, events) without needing direct access to internal service instances or implementation details.

## Key Notes

> [!Summary]+ Key Responsibilities & Features:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">API Abstraction & Facade:</span> Hides internal service implementations, providing a simplified and curated set of functionalities relevant to plugins.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Controlled Access:</span> Defines *what* plugins can access and modify within the application, enforcing boundaries.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Central Entry Point:</span> Acts as the main object through which plugins interact with most of Faseeh's features.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Stability Goal:</span> Aims to provide a relatively stable API across Faseeh versions, minimizing breaking changes for plugin developers (though additions are expected).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Process Bridge Client (`app.storage`):</span> Includes facades (like `app.storage`) that internally use the Preload Script's [[Storage API]] to securely communicate with Main process services via IPC.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Renderer Service Access:</span> Provides facades (`app.plugins`, `app.contentAdapters`, etc.) or direct access (`app.workspaceEvents`) to functionalities managed by Renderer process services.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Construction:</span> Instantiated by the [[Plugin Manager]] service (Renderer) and passed to each [[Base Plugin|BasePlugin]] instance's constructor.

> [!Example]- Interaction & Usage (within a Plugin):
> ```typescript
> // Inside MyPlugin extends BasePlugin
> async onload() {
>   // Access basic app info
>   console.log(`Running on Faseeh v${this.app.appInfo.version}`);
>
>   // Use Storage Facade (triggers IPC)
>   const settings = await this.loadData(); // Uses this.app.storage internally
>   const videos = await this.app.storage.queryMedia({ type: 'video' });
>
>   // Register UI elements
>   this.app.workspace.addCommand({ id: '...', name: '...', callback: () => {} });
>
>   // Register functionality
>   this.app.contentAdapters.register({ id: 'my-adapter', ... }, this.myAdaptFunction);
>
>   // Listen to shared events
>   this.registerEvent(this.app.vaultEvents, 'media:deleted', this.handleMediaDelete);
>
>   // Access other plugins
>   const otherPlugin = this.app.plugins.getPlugin('other-plugin-id');
>   otherPlugin?.somePublicApiMethod();
>
>   // Show notification
>   this.app.notifications.info('MyPlugin loaded!');
> }
> ```

## API Types

### Interface Definition

> [!Summary]- Main Interface : `FaseehApp`
> Defines the structure and methods available on the `app` object passed to plugins.
> ```typescript
> import { EventEmitterWrapper } from './Event Emitter Wrapper'; // Import the wrapper type
> import { BasePlugin } from './Base Plugin'; // Import base plugin type
> import { PluginManifest } from './Plugin Manager'; // Import manifest type
> // Import types for Storage API facade methods, Content Adapters, OCR, Tokenizer, etc.
> // Example placeholder types:
> type StorageAPIFacade = { /* Method signatures matching Storage API */ };
> type ContentAdapterRegistryFacade = { /* Method signatures like register */ };
> type OCRRegistryFacade = { /* Method signatures like register, findBestEngine */ };
> type TokenizerRegistryFacade = { /* Method signatures like register, tokenizeText */ };
> type LanguageDetectorFacade = { /* Method signatures like detectLanguage */ };
> type WorkspaceFacade = { /* Method signatures for UI interaction */ };
> type PluginManagerFacade = { getPlugin(id: string): BasePlugin | null; /* ... */ };
> type NotificationsFacade = { info(msg: string): void; warn(msg: string): void; error(msg: string): void; };
>
> export interface FaseehApp {
>   /** Basic application information */
>   appInfo: {
>     readonly version: string;
>     readonly platform: 'win' | 'mac' | 'linux';
>   };
>
>   /** Facade for interacting with the Storage Service (Main Process) via IPC */
>   storage: StorageAPIFacade;
>
>   /** Facade for interacting with the Plugin Manager (e.g., accessing other plugins) */
>   plugins: PluginManagerFacade;
>
>   /** Facade for registering Content Adapters */
>   contentAdapters: ContentAdapterRegistryFacade;
>
>   /** Facade for registering OCR Engines and finding usable instances */
>   ocrEngines: OCRRegistryFacade;
>
>   /** Facade for registering Text Tokenizers and performing tokenization */
>   tokenizers: TokenizerRegistryFacade;
>
>   /** Facade for detecting language of text */
>   languageDetector: LanguageDetectorFacade; // Added based on LanguageDetector service
>
>   /** Methods for interacting with the application workspace (UI) */
>   workspace: WorkspaceFacade;
>
>   /** Methods for displaying notifications to the user */
>   notifications: NotificationsFacade;
>
>   // --- Shared Event Emitter Instances ---
>   /** Emitter for workspace/UI related events */
>   workspaceEvents: EventEmitterWrapper;
>   /** Emitter for vault/storage related events */
>   vaultEvents: EventEmitterWrapper;
>   /** Emitter for plugin lifecycle related events */
>   pluginEvents: EventEmitterWrapper;
>   // ... potentially other domain-specific emitters ...
> }
> ```

This overview establishes the [[FaseehApp]] object as the well-defined gateway for plugins into the application's ecosystem.