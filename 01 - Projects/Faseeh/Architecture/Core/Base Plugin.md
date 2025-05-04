---
Where ?: Renderer Process
Type: Abstract Base Class
---

# Base Plugin Overview

> [!Note] Description
> The foundational abstract class that all Faseeh community plugins must inherit from. It defines the core lifecycle methods (`onload`, `onunload`) that the `Plugin Manager Service` interacts with and provides essential helper methods and properties (`this.app`, `this.manifest`) for plugin development and enabling **automatic cleanup** of resources, particularly event listeners, when a plugin is unloaded.

## Key Notes

> [!Summary]+ Key Responsibilities & Features:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Lifecycle Contract (Abstract Methods):</span> Defines the mandatory `onload(): Promise<void>` and `onunload(): void | Promise<void>` methods that plugin developers must implement to handle plugin activation and deactivation logic.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">API & Manifest Access:</span> Receives and stores references to the shared [[FaseehApp]] API object (`this.app`) and the plugin's specific `manifest` object (`this.manifest`) passed by the Plugin Manager during instantiation, making them readily available to the plugin's methods.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Event Listener Helper (`registerEvent` method):</span> Provides a **safe and recommended** method for plugins to register listeners on the shared [[Event Emitter Wrapper]] instances (e.g., `this.app.workspaceEvents`). This helper method ensures the listener registration is internally tracked.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Automatic Listener Cleanup:</span> Contains the internal `_trackListenerRegistration` method (called by [[Event Emitter Wrapper]]) and the crucial `_cleanupListeners` method. This `_cleanupListeners` method is **automatically called by the Plugin Manager** *after* the plugin's `onunload` finishes, ensuring all listeners registered via `registerEvent` are reliably removed from the event emitters, preventing memory leaks.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Data Persistence Helpers (`loadData`, `saveData` methods):</span> Offers convenient `async` methods for plugins to load and save their specific configuration or state (`data.json`) using the underlying `Storage API` (`this.app.storage`), automatically scoping the data to the plugin's unique ID.

> [!Example]- Interaction & Usage:
> 1.  **Plugin Definition:** A developer creates `MyPlugin extends Plugin`.
> ---
> 2.  **Instantiation:** The [[Plugin Manager|Plugin Manager]] creates an instance: `new MyPlugin(faseehAppInstance, manifest)`. `this.app` and `this.manifest` are now set.
> ---
> 3.  **Loading:** [[Plugin Manager|Plugin Manager]] calls `await myPluginInstance.onload()`.
> ---
> 4.  **Event Registration (in `onload`):** `MyPlugin` calls `this.registerEvent(this.app.vaultEvents, 'media:saved', this.onMediaSaved)`. The listener is registered, and [[Base Plugin|BasePlugin]] tracks it internally.
> ---
> 5.  **Unloading:** [[Plugin Manager|Plugin Manager]] calls `await myPluginInstance.onunload()` (plugin's custom cleanup runs).
> ---
> 6.  **Automatic Cleanup:** [[Plugin Manager|Plugin Manager]] calls `myPluginInstance._cleanupListeners()`. [[Base Plugin|BasePlugin]] iterates through tracked listeners and removes them from the corresponding [[Event Emitter Wrapper]] instances.
> ---
> 7. **Data Persistence:** Plugin calls `await this.loadData()` or `await this.saveData(settings)` to manage its state.

> [!Tip] Core Benefit:
> The primary benefit of extending [[Base Plugin|BasePlugin]] (beyond enforcing structure) is the **guaranteed automatic cleanup of event listeners**, significantly reducing the risk of memory leaks and zombie listeners, which are common problems in complex plugin systems. Developers focus on their `onload` and `onunload` logic, relying on the base class for this critical housekeeping task.

## API Types

### Core Properties Provided to Plugins

> [!Summary]- Application API : [[FaseehApp]]
> The primary API object passed to the plugin's constructor (`this.app`). Provides access to core application functionalities, shared event emitters, and facades for interacting with services (like storage via IPC). Its specific structure is defined elsewhere but includes modules like `app.storage`, `app.plugins`, `app.workspace`, `app.workspaceEvents`, etc.
> ```typescript
> export declare type FaseehApp = { /* ... API methods and properties ... */ };
> ```

![[Plugin Manager#^9cf241|Plugin Manifest]]
### Core Methods Provided by Base Class

> [!Summary]- Helper Method : `registerEvent`
> Used by plugins within `onload` (or other methods) to safely register event listeners with automatic cleanup.
> ```typescript
> import { EventEmitterWrapper } from './Event Emitter Wrapper'; // Assuming definition
> import { EventType, Handler } from 'mitt'; // Or underlying emitter types
>
> registerEvent<T = any>(
>     emitterWrapper: EventEmitterWrapper, // e.g., this.app.vaultEvents
>     eventName: EventType,
>     callback: Handler\<T\>
> ): void;
> ```

> [!Summary]- Helper Method : `loadData`
> Used by plugins to load their specific persistent data (`data.json`). Internally uses `this.app.storage.loadPluginData`.
> ```typescript
> loadData(): Promise\<any>;
> ```

> [!Summary]- Helper Method : `saveData`
> Used by plugins to save their specific persistent data (`data.json`). Internally uses `this.app.storage.savePluginData`.
> ```typescript
> saveData(data: any): Promise\<void>;
> ```

### Core Methods Implemented by Plugins

> [!Summary]- Abstract Method : `onload`
> Must be implemented by the plugin subclass. Contains the plugin's activation logic.
> ```typescript
> onload(): Promise\<void>;
> ```

> [!Summary]- Abstract Method : `onunload`
> Must be implemented by the plugin subclass. Contains the plugin's custom deactivation/cleanup logic, executed *before* automatic listener cleanup.
> ```typescript
> onunload(): void | Promise\<void>;
> ```

