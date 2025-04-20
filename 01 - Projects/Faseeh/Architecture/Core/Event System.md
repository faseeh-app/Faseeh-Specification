---
Where ?: Renderer Process
---
# Event System Overview

> [!Note] Description
> Provides a publish-subscribe mechanism enabling different parts of the application (core services, UI components, and especially plugins) to announce that something significant has happened (an "event") and allowing interested parties ("listeners" or "subscribers") to react accordingly without direct coupling.

## Key Notes

> [!Summary] Key Components & Concepts 
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Event Emitter/Bus :</span> A central instance (e.g., using Node.js `EventEmitter`, `mitt`, or similar library) responsible for registering listeners and dispatching events. This instance is typically managed internally, accessible via the `FaseehApp` API. 
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Event Names/Types :</span> Clearly defined, often namespaced strings identifying specific occurrences (e.g., `'media:opened'`, `'storage:media-saved'`, `'plugin:loaded'`, `'workspace:layout-change'`). 
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Event Payload :</span> Data associated with an event, passed to listeners when the event is dispatched (e.g., the ID of the opened media, the saved `MediaObject`, the ID of the loaded plugin). 
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Listener Registration (`on`/`subscribe`) :</span> Plugins (and core components) use methods exposed via the `FaseehApp` API (e.g., `app.storage.on(...)`, `app.sync.on(...)`, or a generic `app.events.on(...)`) to register callback functions that should be executed when a specific event occurs. 
> > <span style="font-weight:bold; color:rgb(255, 192, 0)">Crucial :</span> The `Plugin` base class should provide helper methods (e.g., `this.registerEvent(eventEmitter, eventName, callback)`) that automatically handle unregistration during `onunload`. 
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Listener Unregistration (`off`/`unsubscribe`) :</span> Mechanisms to remove listeners, primarily handled automatically by the base `Plugin` class's cleanup during `onunload` to prevent memory leaks. Manual unregistration might also be exposed. 
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Event Dispatching (`emit`/`publish`) :</span> Core services or components trigger events by calling an internal `emit` method on the central emitter when a relevant action completes (e.g., after saving a file, after opening media, after a plugin loads). 

> [!Example]- Workflow (Plugin Listening):
> 1.  **Registration (Plugin `onload`)**: Plugin calls `this.registerEvent(app.workspace, 'media:opened', this.handleMediaOpen)`. This registers `this.handleMediaOpen` with the event bus *and* records the registration for later cleanup.
> ---
> 2.  **Event Occurs (Core Service/UI)**: User opens a media file. The relevant component (e.g., Workspace manager) emits the `'media:opened'` event with the media ID: `app.workspace.trigger('media:opened', mediaId)`. *(Using `trigger` as an example public emit method)*
> ---
> 3. **Dispatch (Event Bus)**: The central event bus finds all registered listeners for `'media:opened'`.
> ---
> 4.  **Execution (Plugin):** The plugin's `this.handleMediaOpen(mediaId)` function is called with the media ID.
> ---
> 5.  **Cleanup (Plugin `onunload`):** When the plugin is disabled, the base `Plugin` class's `onunload` logic automatically calls the unregistration method on the event bus for all events registered using `this.registerEvent`.