---
Where ?: Renderer Process
Type: Utility Class
---

# Listener Tracker Overview

> [!Note] Description
> A helper class designed for **core Renderer services and UI components** (which do not inherit from [[Base Plugin|BasePlugin]]) to manage the lifecycle of event listeners they register on the shared [[Event Emitter Wrapper]] instances. It provides a centralized way to track registrations and ensure they are properly cleaned up when the owning service or component is destroyed, preventing memory leaks.

#### Key Notes

> [!Summary]+ Key Responsibilities:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Listener Tracking:</span> Provides a `register` method that takes an [[Event Emitter Wrapper]] instance, event name, and callback. It registers the listener with the underlying emitter **and** stores the registration details internally within the tracker instance.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Centralized Cleanup (`cleanupAll` method):</span> Offers a `cleanupAll` method that iterates through all tracked listener registrations and calls the appropriate `off` method on the corresponding underlying emitter for each one, effectively unregistering them.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Manual Initiation:</span> Unlike the automatic cleanup in [[Base Plugin|BasePlugin]], the cleanup process for listeners managed by [[Listener Tracker]] **must be explicitly initiated** by calling its `cleanupAll` method.
> - ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Scope:</span> Each instance of [[Listener Tracker]] manages listeners for the specific core service or UI component instance that creates and owns it.

> [!Example]- Interaction & Usage:
> 1.  **Instantiation:** A core Renderer service (e.g., `ContentAdapterManager`) creates its own instance: `this.listenerTracker = new ListenerTracker();`.
> ---
> 2.  **Registration:** The service uses the tracker to register listeners on shared [[Event Emitter Wrapper]] instances (which it receives via DI or context): `this.listenerTracker.register(this.context.pluginEvents, 'plugin:loaded', this.handlePluginLoaded);`.
> ---
> 3.  **Operation:** Listeners execute normally when events are emitted on the shared wrappers.
> ---
> 4.  **Cleanup Trigger:** At a well-defined point when the service is no longer needed (e.g., application shutdown, window closing), the service's own `destroy()` or cleanup method **must call** `this.listenerTracker.cleanupAll();`.
> ---
> 5.  **Unregistration:** The `cleanupAll` method iterates through its tracked registrations and removes each listener from the underlying emitter.

> [!Tip] Key Difference from [[Base Plugin|BasePlugin]]:
> - [[Base Plugin|BasePlugin]] provides **automatic** cleanup linked to the plugin unload lifecycle, managed by the Plugin Manager.
> - [[Listener Tracker]] requires **manual initiation** of cleanup by the owning service/component code at the appropriate time.

> [!warning] Important Consideration:
> It is crucial to ensure that the `cleanupAll` method of every [[Listener Tracker]] instance is reliably called during the application's teardown phase to prevent listener leaks from core services.