---
Type: File
---
# `source.[ext]` File Overview

> [!Note] Description
> Represents the primary source file associated with a `MediaObject`, potentially copied into the library during import. This could be the original video file, audio file, PDF, EPUB, etc. The actual filename might be standardized (e.g., always `source` with the correct extension) or retain the original name.
## Key Notes

> [!Summary]+ Content:
> - Binary or text content of the originally imported file.
> ---
> - **Managed By:** [[Storage Service]] (Main Process). Created/copied during the import process orchestrated by the [[Content Adapter Registry]]. Its path is typically stored in the corresponding `MediaObject` database record (`sourceStoragePath` or similar). Read operations might be initiated by the Renderer (e.g., for playback) via the Storage API. Deleted when the parent `MediaObject` is deleted.
> ---
> - **Optionality:** Depending on the import strategy and content type, the application might store a reference to the original external file (`sourceUri` in `MediaObject`) instead of copying it internally, in which case this file might not exist. Storing internally increases robustness but also disk usage.

> [!Tip] Purpose:
> Allows Faseeh to access the core media content reliably, even if the user moves or deletes the original file externally (if copied internally). Used for playback (video/audio) or potentially re-processing by adapters.