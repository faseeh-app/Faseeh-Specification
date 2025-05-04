---
Type: Folder
---
# `models` Directory Overview

> [!Note] Description
> Stores downloaded Artificial Intelligence / Machine Learning models used by various Faseeh features, such as Speech-to-Text (for subtitle generation) or Optical Character Recognition (OCR). Keeping models within the `.faseeh` data directory allows for user-managed or dynamically downloaded models separate from the main application bundle.
## Key Notes

> [!Summary]+ Content:
> - Subdirectories or files representing specific AI/ML models (e.g., Whisper model files, Tesseract language data files).
> - Structure depends on the requirements of the specific model libraries being used.
> ---
> - **Managed By:** Potentially managed by specific services responsible for features using these models (e.g., [[OCR Registry Service]], a future Subtitle Generation Service). These services might handle downloading, verifying, and providing paths to these models. The [[Storage Service]] would handle the actual file storage/path retrieval based on requests from these managing services.
> ---
> - **Access:** Paths to required models are likely configured or discovered by the services that need them (e.g., `OCREngine` implementations).

> [!Tip] User Control:
> This structure allows for potentially large model files to be managed within the user's data directory rather than bloating the core application installation. Users might be given options to download/manage models.