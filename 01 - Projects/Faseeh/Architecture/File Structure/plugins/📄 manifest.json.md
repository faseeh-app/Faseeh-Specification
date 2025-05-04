---
Type: File
---
# `manifest.json` (Plugin) Overview

> [!Note] Description
> A required JSON file within each plugin's directory that provides essential metadata about the plugin to Faseeh. It defines the plugin's identity, version, compatibility, entry point, and dependencies *(not node dependencies, but rather other plugins as dependecies)*

## Key Notes

> [!Summary]+ Content:
> - A JSON object adhering to the `PluginManifest` interface.
> - **Required Fields:** `id`, `name`, `version`, `minAppVersion`, `main`.
> - **Recommended Fields:** `description`, `author`, `pluginDependencies`.
> - *(See [[Plugin Manager#^9cf241|Plugin Manifest Definition]] for full structure)*.
> ---
> - **Managed By:** Created by the plugin developer. Included when the plugin is packaged/installed.
> ---
> - **Read By:** [[Plugin Manager]] during plugin discovery and loading to determine compatibility, load order, and entry point.

