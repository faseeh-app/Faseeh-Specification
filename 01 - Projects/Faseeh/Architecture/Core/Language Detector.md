---
Where ?: Renderer Process
Type: Core Service
---
# Language Detector Service Overview

> [!Note] Description
> Responsible for analyzing text input to determine its most likely language. It encapsulates language detection logic and models, providing a consistent and reusable service for other components (like Content Adapters or UI elements) that need to identify the language of user-provided or extracted text where it's not explicitly defined.

## Key Notes

> [!Summary]+ Key Responsibilities:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Language Detection Logic:</span>Implements or wraps a language detection library (e.g., `franc`) to analyze input text strings.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Model Management:</span> Manages the loading and accessibility of any language models required by the chosen detection library.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Detection API (`detectLanguage` method):</span> Exposes a primary asynchronous method that takes text and returns the detected ISO 639 language code (or null if detection fails/is uncertain), along with a confidence score.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Configuration (Optional):</span> May provide ways to configure detection parameters, such as minimum text length for reliable detection or whitelisting/blacklisting specific languages.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">API Exposure (via [[FaseehApp]]  or DI):</span> Makes its `detectLanguage` method available to other Renderer services and potentially plugins via Dependency Injection or a facade on the [[FaseehApp]] object (e.g., `app.languageDetector.detectLanguage(...)`).

> [!Example]- Workflow (Content Adapter using Language Detector):
> 1.  **Adapter Receives Text:** A [[Content Adapter]] processes a source (e.g., pasted text, plain `.txt` file) where language is unknown.
> ---
> 2.  **Adapter Calls Detector:** Adapter calls 
>    `const detectionResult = await this.languageDetector.detect(extractedText);`
> ---
> 3.  **Detector Analyzes:** The `Language Detector Service` uses its internal library/model to analyze the text.
> ---
> 4.  **Detector Returns Result:** Service returns `{ languageCode: 'fr', confidence: 0.92 }`.
> ---
> 5.  **Adapter Uses Result:** The [[Content Adapter]] populates the `mediaObjectData.language` field with `'fr'` before passing the result to the [[Content Adapter Registry]] for storage coordination.

> [!Tip] Integration:
> - Used primarily by [[Content Adapter]] during import processing.
> - May also be used by UI components or other services that handle raw text input without pre-defined language context.
> - Could potentially be made pluggable itself (allowing different detection engines) via a registry pattern, but starting with a single core implementation is likely sufficient.

> [!warning] Limitations:
> Language detection accuracy varies, especially for short texts or mixed-language content. Confidence scores should be considered by providing for example users with the ability to override or confirm the detected language.
