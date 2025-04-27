---
Where ?: Renderer Process
Type: Abstract Base Class
---

# BasePlugin Overview

> [!Note] Description
> The foundational abstract class that all Faseeh community plugins must inherit from. It defines the core lifecycle methods (`onload`, `onunload`) that the `Plugin Manager Service` interacts with and provides essential helper methods and properties (`this.app`, `this.manifest`) for plugin development and enabling **automatic cleanup** of resources, particularly event listeners, when a plugin is unloaded.

## Key Notes

> [!Summary]+ Key Responsibilities & Features:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Lifecycle Contract (Abstract Methods):</span> Defines the mandatory `onload(): Promise<void>` and `onunload(): void | Promise<void>` methods that plugin developers must implement to handle plugin activation and deactivation logic.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">API & Manifest Access:</span> Receives and stores references to the shared `FaseehApp` API object (`this.app`) and the plugin's specific `manifest` object (`this.manifest`) passed by the Plugin Manager during instantiation, making them readily available to the plugin's methods.
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
> 4.  **Event Registration (in `onload`):** `MyPlugin` calls `this.registerEvent(this.app.vaultEvents, 'media:saved', this.onMediaSaved)`. The listener is registered, and [[🧩Faseeh Plugin|BasePlugin]] tracks it internally.
> ---
> 5.  **Unloading:** [[Plugin Manager|Plugin Manager]] calls `await myPluginInstance.onunload()` (plugin's custom cleanup runs).
> ---
> 6.  **Automatic Cleanup:** [[Plugin Manager|Plugin Manager]] calls `myPluginInstance._cleanupListeners()`. [[🧩Faseeh Plugin|BasePlugin]] iterates through tracked listeners and removes them from the corresponding [[Event Emitter Wrapper]] instances.
> ---
> 7. **Data Persistence:** Plugin calls `await this.loadData()` or `await this.saveData(settings)` to manage its state.

> [!Tip] Core Benefit:
> The primary benefit of extending [[🧩Faseeh Plugin|BasePlugin]] (beyond enforcing structure) is the **guaranteed automatic cleanup of event listeners**, significantly reducing the risk of memory leaks and zombie listeners, which are common problems in complex plugin systems. Developers focus on their `onload` and `onunload` logic, relying on the base class for this critical housekeeping task.