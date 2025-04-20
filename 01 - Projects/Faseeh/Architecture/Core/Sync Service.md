# Overview
Sync service is responsible for synchronizing data between the local device and the cloud. It ensures that the data is up-to-date and consistent across all devices.
## Key Notes
- The service can listen for changes in the local storage and push those changes to the cloud.
- It can also pull changes from the cloud upon initialization and in regular intervals.
- The service can have multiple providers, each responsible for a different cloud storage solution (e.g., Google Drive, Dropbox, etc.).
- The service emits events that can trigger the [[Storage Service]] to update the local storage.
- The service interface is defined as follows:
```ts
interface SyncService {
  init(): Promise<void>; // Initialize a listener and load providers
  push(localPaths: string[]): Promise<void>; // Push local changes to the cloud
  pull(remotePaths: string[]): Promise<void>; // Pull remote changes from the cloud
  close(): Promise<void>; // Close the service and stop listening for changes
  // There can many other methods likes exporting, logging, etc.
}
```
- Providers can be addeed through plugins, that implement the SyncProvider interface.
```ts
abstract class SyncProvider {
  abstract name: string; // e.g., "Google Drive", "Dropbox", "Faseeh Backend"
  abstract connect(config: SyncConfig): Promise<void>;
  abstract push(localPaths: string[]): Promise<void>;
  abstract pull(remotePaths: string[]): Promise<void>;
  abstract resolveConflicts(): Promise<SyncResolution>; // Resolve conflicts between local and remote changes
}
```

> [!Note]
> Arised conflicts are the plugin's responsibility to resolve. it can be simply a last write wins strategy, an interactive UI, or even complex custom logic as merging.