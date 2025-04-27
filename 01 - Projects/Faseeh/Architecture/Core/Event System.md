---
Where ?: Renderer Process
---
# Event System Overview

> [!Note] Description
> Provides a publish-subscribe mechanism enabling different parts of the application (core services, UI components, and especially plugins) to announce that something significant has happened (an "event") and allowing interested parties ("listeners" or "subscribers") to react accordingly without direct coupling.

## Key Notes

> [!Summary]+ Key Components & Concepts:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Shared Domain Emitters:</span> Instead of one global bus or per-service emitters, the system uses **multiple, domain-specific event emitters** (e.g., `workspaceEvents`, `vaultEvents`, `pluginEvents`). Each is an instance of the [[Event Emitter Wrapper]] utility class. These shared instances are created centrally during application setup.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Event Names/Types:</span> Clearly defined strings identifying specific occurrences within a domain (e.g., `workspaceEvents` emits `'media:opened'`, `vaultEvents` emits `'media:saved'`).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Event Payload:</span> Data associated with an event, passed to listeners (e.g., the saved `MediaObject`, the ID of the loaded plugin).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Listener Registration:</span>
>     - **Plugins:** Use helper methods (e.g., `this.registerEvent(...)`) provided by their [[🧩Faseeh Plugin|BasePlugin class]]. These helpers register listeners on the appropriate **shared emitter instance** (accessed via `this.app`) and enable automatic cleanup.
>     - **Core Services/UI:** Register listeners directly on the **shared emitter instances** (received via Dependency Injection). 
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Listener Cleanup Mechanisms:</span> to prevent memory leaks:
>     - **Plugins:** Handled **automatically** by the [[🧩Faseeh Plugin|BasePlugin]] during its `unload` sequence, using internal tracking initiated by the `registerEvent` helper.
>     - **Core Services:** Require **explicit cleanup** using the [[Listener Tracker]] utility class to manage registrations and call `cleanupAll()` during service destruction.
> 	- **UI:** Manual `off()` calls within framework lifecycle hooks (e.g., `onUnmounted` in Vue).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Event Dispatching (`emit`):</span> Core services (having received a shared emitter instance via DI) call the `emit` method on the appropriate **shared emitter** when a relevant action completes.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Core Utilities: This system relies on specific utility components:</span>
> 	- [[Event Emitter Wrapper|EventEmitterWrapper]]: The class implementing the shared emitters, wrapping a library like `mitt` and facilitating the plugin cleanup linkage.
> 	- [[🧩Faseeh Plugin|Plugin]]: The base class for all plugins, providing `registerEvent` helpers and automatic listener cleanup logic.
> 	- [[Listener Tracker|ListenerTracker]]: A utility class used by core services to manage listener cleanup for their own registrations on the shared emitters.

> [!Question]- Why wrap the underlying event emitter library?
> Wrapping (using [[Event Emitter Wrapper|EventEmitterWrapper]]) provides:
> - **Automatic Plugin Cleanup:** Enables [[🧩Faseeh Plugin|BasePlugin]] to reliably track and unregister listeners.
> - **API Abstraction & Stability:** Creates a stable API for Faseeh, allowing internal emitter library changes without breaking plugins/services.
> - **Controlled Interface:** Allows defining exactly how events are registered and potentially restricting emission.

> [!Example]- Workflow (Plugin Listening):
> 1.  **Registration:** Plugin's `onload` calls `this.registerEvent(this.app.vaultEvents, 'media:saved', this.onMediaSaved)`. Wrapper registers listener with `vaultEvents`' internal emitter and tracks it via [[🧩Faseeh Plugin|BasePlugin]].
> ---
> 2.  **Event Occurs:** Core service calls `this.vaultEvents.emit('media:saved', mediaData)` after saving.
> ---
> 3.  **Execution:** Plugin's `this.onMediaSaved(mediaData)` method is executed.
> ---
> 4.  **Cleanup:** Plugin Manager unloads plugin, calls `plugin.onunload()`, then calls internal cleanup which automatically calls `vaultEvents.emitter.off('media:saved', this.onMediaSaved)`.

> [!Example]- Workflow (Core Service Listening):
> 1.  **Registration:** `ContentAdapterManager` (using [[Listener Tracker]]) calls `this.listenerTracker.register(this.pluginEvents, 'plugin:loaded', this.handlePluginLoaded)`. Tracker registers listener with `pluginEvents`' internal emitter and records it locally.
> ---
> 2.  **Event Occurs:** `PluginManagerService` calls `this.pluginEvents.emit('plugin:loaded', pluginId)`.
> ---
> 3.  **Execution:** `ContentAdapterManager`'s `this.handlePluginLoaded(pluginId)` method is executed.
> ---
> 4.  **Cleanup:** During app shutdown, `ContentAdapterManager.destroy()` is called, which calls `this.listenerTracker.cleanupAll()`, which calls `pluginEvents.emitter.off('plugin:loaded', this.handlePluginLoaded)`.

> [!Example]- Workflow (Event Originating from Main Process):
> 1.  **Detection (Main):** [[Storage Service]] detects external change.
> ---
> 2.  **IPC Send (Main -> Renderer):** Sends IPC message `storage-external-change` with details.
> ---
> 3.  **IPC Reception (Renderer):** Core component (e.g., dedicated IPC bridge or Storage Facade) receives the message.
> ---
> 4.  **Renderer Emit (Shared Emitter):** The receiver calls `this.vaultEvents.emit('external-change', details)`.
> ---
> 5.  **Execution (Renderer):** Plugins and core services listening for `'external-change'` on `vaultEvents` are notified.

> [!warning]- Integration with FaseehApp API:
> The `FaseehApp` object provided to plugins primarily exposes the **shared [[Event Emitter Wrapper]] instances** for registering listeners:
> - `app.workspaceEvents.on(...)`
> - `app.vaultEvents.on(...)`
> - `app.pluginEvents.on(...)`
> - *(etc.)*

