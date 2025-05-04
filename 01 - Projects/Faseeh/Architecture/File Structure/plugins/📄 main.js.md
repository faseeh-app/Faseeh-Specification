---
Type: File
---
# `main.js` (Plugin) Overview

> [!Note] Description
> The main JavaScript entry point for a plugin, as specified by the `main` field in its `manifest.json`. This file is typically the **output of a build process** (e.g., using Vite, Rollup, Webpack) which compiles the plugin's source code (often TypeScript) and **bundles its required Node.js dependencies** into a distributable format. It must export a default class that extends the [[Base Plugin]] abstract class.

## Key Notes

> [!Summary]+ Content:
> - **Bundled JavaScript Code:** Contains the compiled and bundled JavaScript code for the entire plugin, including code written by the plugin developer **and code from any third-party Node.js libraries** the plugin depends on.
> ---
> - **Entry Point:** Serves as the single file loaded by Faseeh's [[Plugin Manager]] to start the plugin.
> ---
> - **`BasePlugin` Export:** **Must** export a default class `export default class MyPlugin extends BasePlugin { ... }`.
> ---
> - **Core Logic:** Contains the plugin's implementation logic within the exported class, including the `onload()` and `onunload()` methods, event listener registrations, command additions, API registrations (adapters, etc.), and UI interactions.
> ---
> - **Managed By:** Created by the plugin developer's build process. Included during installation.
> ---
> - **Executed By:** Loaded and executed by the [[Plugin Manager]] using Node.js `require()` when the plugin is enabled and loaded.

> [!Tip] Plugin Dependencies:
> The bundling step performed by the plugin developer is essential. It allows plugins to use `npm` packages without requiring Faseeh users to run `npm install` inside the plugin folders. All necessary code is packaged into `main.js` (and potentially related chunks if code splitting is used).