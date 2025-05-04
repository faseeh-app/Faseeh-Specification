---
Where ?: Renderer Process
Type: Utility Class
---
# Event Emitter Wrapper Overview

> [!Note] Description
> A wrapper class built around a standard event emitter library (e.g., `mitt`). It standardizes event handling (`on`, `off`, `emit`) and is a key component enabling **automatic listener cleanup** for plugins when used in conjunction with the [[Base Plugin|BasePlugin]] class. Multiple instances are typically created to manage events for different application domains (e.g., workspace, vault).

> [!Question]- Why wrap the underlying event emitter library?
> - **Automatic Plugin Cleanup:** Enables [[Base Plugin|BasePlugin]] to reliably track and unregister listeners.
> - **API Abstraction & Stability:** Creates a stable API for Faseeh, allowing internal emitter library changes without breaking plugins/services.
> - **Controlled Interface:** Allows defining exactly how events are registered and potentially restricting emission.
## Key Notes

> [!Summary]+ Key Responsibilities:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Emitter Encapsulation:</span> Abstracts the specific underlying event emitter library (`mitt`), providing a stable interface.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Plugin Cleanup Facilitation (`on` method):</span> Its primary `on` method **requires the listener's owner** (a [[Base Plugin|BasePlugin]] instance) as an argument. It registers the listener with the internal emitter **and** calls the owner's internal `_trackListenerRegistration` method, enabling the [[Base Plugin|BasePlugin]] to automatically unregister the listener during unload.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Controlled Event Emission (`emit` method):</span> Provides a method to dispatch events on its specific internal emitter. This is typically used by core services to announce state changes within the wrapper's domain.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Manual Unregistration (`off` method):</span> Offers a standard `off` method for cases where manual listener removal might be needed (though less common for plugins using the automatic cleanup).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Domain Scoping:</span> Each instance manages events for a specific application area (e.g., `workspaceEvents`, `vaultEvents`), providing logical separation.

> [!Example]- Interaction:
> 1.  **Instantiation:** Several instances (e.g., `workspaceEvents`, `vaultEvents`) are created during Renderer process setup.
> ---
> 2.  **Core Services:** Receive instances via Dependency Injection. They call `emit` to dispatch events and `on` (typically using a [[Listener Tracker|ListenerTracker]] for cleanup) to subscribe.
> ---
> 3.  **Plugin API ([[FaseehApp]]):** Instances are exposed on the [[FaseehApp]] object (e.g., `app.workspaceEvents`).
> ---
> 4.  **Plugins:** Use [[Base Plugin|BasePlugin]]'s `registerEvent(this.app.workspaceEvents, ...)` helper. This helper calls the wrapper's `on` method, passing `this` (the plugin instance) as the owner, enabling automatic cleanup. Plugins generally do not call `emit`.
> ---
> 5.  **Cleanup:** Listener removal is handled automatically by [[Base Plugin|BasePlugin]] (for plugins) or explicitly via [[Listener Tracker|ListenerTracker]] / manual calls (for core services).
