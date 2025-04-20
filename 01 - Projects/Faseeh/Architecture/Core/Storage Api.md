---
Where ?: Preload Script
---
# Storage API Overview

> [!Note] Description
> Provides a secure bridge for the Renderer process to access the Storage Service in the Main process. It ensures that all storage operations are performed securely and efficiently, while also providing a simplified interface for Renderer components (UI, Plugins) to interact with the Storage Service.

> [!Question]- Why do we need this if Renderer process can use `fs` directly ?
>Since `nodeIntegration = true` and `contextIsolation = false`, renderer process, including plugins, can access `fs` directly. However, it's better to use this API for the following reasons:
> - In case we find a way to isolate plugins while keeping the flexibility of accessing npm packages, then this API will be the only way to access storage, therefore we will avoid breaking changes in this aspect.
> - Storing operations passing through the API can benefit from additional processing features like validation, logging, metadata management, etc.
## Key Notes

- **Renderer Process Interface:** Exposes a simplified API for the Renderer process to interact with the Storage Service.
- **IPC Communication:** Uses Electron's `ipcRenderer` to send requests to the Main process and receive responses.

> [!Todo] Todo: Define the API interface

