---
Where ?: Renderer Process
---
# Plugin Manager Service Overview

> [!Note] Description
> Responsible for the complete lifecycle of community plugins. It discovers, loads, manages, and unloads plugins, provides them with the necessary API access (`FaseehApp` object, manifest object), and ensures they integrate correctly (and safely, as much as possible) into the application.

> [!Question]- Why do we need to pass the manifest object to the plugin instance?
> The manifest object contains essential metadata about the plugin, such as its ID, name, version, description, author, and any other relevant information. This allows the plugin to access its own metadata and use it as needed during its operation. It also helps in validating the plugin's compatibility with the application.
## Key Notes

> [!Summary]+ Key Responsibilities:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Plugin Discovery:</span> Scan the designated plugin directory (`.faseeh/plugins/`) at startup (and potentially on command) to identify installed plugins by reading their manifest.json files. Uses [[Storage API]] (`faseehStorageAPI.getAppPath('plugins')`) to find the directory.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">State Management:</span> Read and maintain the list of user-enabled plugins (from `config/enabled-plugins.json` via the [[Storage API]]). Provide methods to enable/disable plugins, updating this configuration.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Dependency Resolution & Loading Order:</span> 
> 	 - Read the `pluginDependencies` field (an array of plugin IDs, potentially with version constraints later) from each plugin's `manifest.json`.
> 	 - Determine a valid loading order based on dependencies (e.g., using topological sort on the dependency graph). Ensure dependencies are loaded *before* the plugins that require them.
> 	 - Refuse to load plugins whose dependencies are missing, disabled, incompatible (`minAppVersion`), or failed to load themselves.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Plugin Loading:</span> For each plugin that is valid (installed, enabled, and compatible `minAppVersion`) and has all dependencies loaded:
> 	- Load the plugin's main code file (main.js specified in manifest) using Node.js `require()` (available in Renderer since `nodeIntegration` is enabled).
> 	- Instantiate the plugin's main class, passing it the crucial `FaseehApp` API object and its manifest.
> 	- Call the plugin instance's asynchronous `onload()` method.
> 	- Store and manage the active plugin instance in the internal registry.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Plugin Unloading:</span> When a plugin is disabled or the application is closing:
> 	- Retrieve the active plugin instance.
> 	- Call the plugin instance's `onunload()` method to allow for cleanup.
> 	- Immediately after onunload completes (or fails), call the internal `pluginInstance._cleanupListeners()` method to automatically unregister all event listeners.
> 	- Remove the instance from the internal registry.
> <span style="color:rgb(255, 192, 0)">⚠️ **Consideration :**</span> If a core dependency is unloaded, consider the impact on dependent plugins. It may be simpler to require a restart of the application.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">API Provision (FaseehApp):</span> Construct and provide the `FaseehApp` API object to each loaded plugin. This object acts as the plugin's interface to Faseeh's core functionalities, wrapping direct calls to Renderer services (like [[Content Adapter Registry]]) and using the Preload [[Storage API]] for secure access to Main process services (like [[Storage Service]]). Crucially includes `plugin.loadData()` and `plugin.saveData()` methods which internally use the Storage API to manage the plugin's specific `data.json`.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Error Handling:</span> Gracefully handle errors during plugin discovery, loading, execution (`onload`/`onunload`), and unloading. Log errors, notify the user, and potentially automatically disable faulty plugins to prevent application instability.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Metadata Access:</span> Provide methods for the UI (e.g., Settings screen) to retrieve information about installed plugins (name, version, description from manifest), their enabled status, and any encountered errors.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Inter-Plugin Access & Registry:</span>
>     - Maintain an internal **registry (e.g., a Map)** mapping loaded plugin IDs to their active class instances.
>     - Expose a method via the `FaseehApp` API object allowing plugins to retrieve instances of other loaded plugins, e.g., `app.plugins.getPlugin(pluginId: string): Plugin | null`.
>     - **Crucially depends on correct dependency declaration:** Plugins should declare dependencies on any plugin they intend to retrieve via `getPlugin`. The Plugin Manager ensures the dependency is loaded first.
>     - **API Exposure:** While `getPlugin` returns the raw instance, i think plugins should expose a cleaner, dedicated API via public methods on their class (e.g., `pluginInstance.getPublicAPI()`) rather than relying on direct access to internal properties/methods.

> [!Example]- Workflow (Loading Enabled Plugins at Startup):
> 1. **Get Paths:** Request plugins directory path and config file path from Main process via [[Storage Api]].
> ---
> 2. **Read Config:** Read the `enabled-plugins.json` file via [[Storage Api]].
> ---
> 3. **Scan Directory:** Scan the plugins directory filesystem using [[Storage Api]].
> ---
> 4. **Read Manifests:** For each subdirectory, read `manifest.json`. Store manifest data.
> ---
> 5. **Build Dependency Graph:** Construct a graph representing dependencies between all *potentially loadable* (installed & compatible) plugins.
> ---
> 6. **Filter Enabled & Check Cycles:** Identify plugins listed in `enabled-plugins.json` and that its `current version` >= `minAppVersion` in manifest. Detect and report critical errors like circular dependencies.
> ---
> 7. **Determine Load Order:** Perform a topological sort on the dependency graph of enabled, valid plugins to get a safe loading sequence.
> ---
> 8. **Load & Initialize:** For each valid, enabled plugin:
> 	- `require()` the plugin's `main.js`.
> 	- Create `FaseehApp` API object instance.
> 	- Instantiate the plugin class (`new PluginClass(faseehApp, manifest)`).
> 	- **Store `pluginInstance` in the internal registry (e.g., `activePlugins.set(manifest.id, pluginInstance)`).**
> 	- Call `await pluginInstance.onload()`.
> 	- **If loading fails, mark this plugin as failed and prevent any plugins depending on it from loading.**
> ---
> 9. **Ready:**  Service signals that initial plugin loading is complete.

> [!Example]- Workflow (Disabling a Plugin via UI):
> 1. **Request:** User clicks "Disable" in the UI for a specific plugin ID.
> ---
> 2. **Dependency Check (Optional):** Optionally warn the user if other enabled plugins depend on this one. Disabling might break dependents.
> ---
> 3. **Find Instance:** Plugin Manager retrieves the active instance for that plugin ID.
> ---
> 4. **Unload:** Call `pluginInstance.onunload()`. and then `pluginInstance._cleanupListeners()`.
> ---
> 5.  **Remove Instance:** Remove the instance from the internal registry (e.g., activePlugins.delete(pluginId)).
> ---
> 6.  **Update Config:** Update the enabled-plugins.json file (remove plugin ID) via faseehStorageAPI.
> ---
> 7.  **UI Update:** Signal UI that the plugin is now disabled.
> ---
> <span style="color:rgb(255, 192, 0)">⚠️ **Consideration :**</span> Dependent plugins might now fail on next load/reload unless also disabled.

> [!Question]- Why call _cleanupListeners after onunload?
> 1. **Plugin Control:** The plugin's onunload method might need to perform actions that rely on its listeners still being active (e.g., emitting one final event, checking some state based on a listener's side effect).
> ---
> 2. **Orderly Shutdown:** It allows the plugin's custom logic to run first, followed by the guaranteed, automatic cleanup provided by the base class.