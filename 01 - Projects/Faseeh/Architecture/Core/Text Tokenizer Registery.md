---
Where ?: Renderer Process
Type: Core Service
---
# Text Tokenizer Registery Overview

> [!Note] Description
> Manages all available text tokenization strategies within Faseeh. It maintains a registry of tokenizers (both built-in and plugin-provided), selects the most appropriate one based on the text's language, and provides a unified interface for other application components to break raw text down into an array of standardized `Token` objects (words, punctuation, etc.) with positional information. This enables language-specific handling, crucial for features like interactive text and dictionary lookups.

#### Key Notes

> [!Summary]+ Key Responsibilities:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Tokenizer Registry Management:</span> Maintains an internal collection of all registered tokenizer implementations, indexed by unique IDs and associated with metadata (supported languages, name, id, ...).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Registration Interface (`via FaseehApp`):</span> Exposes a registration function (`app.tokenizers.register`) allowing the core application (for defaults) and plugins (via [[🧩Faseeh Plugin|BasePlugin]]) to add new tokenizers and specify which languages they handle and their priority.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Core Fallback Tokenizer:</span> Includes at least one built-in, low-priority, language-agnostic tokenizer (e.g., whitespace/punctuation-based) to ensure basic functionality for any language.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Best Tokenizer Selection:</span> Implements logic to determine the most suitable registered tokenizer for a given input language code, prioritizing language-specific implementations and respecting registration priority levels.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Tokenization Execution API:</span> Provides a primary asynchronous method (e.g., `tokenizeText(text: string, languageCode: string): Promise<Token[]>`) that selects and executes the best tokenizer, returning a standardized array of `Token` objects (each containing `text`, `startIndex`, `endIndex`, `isWord`).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Error Handling:</span> Manages scenarios where no suitable tokenizer is found or when a selected tokenizer fails during execution (potentially falling back to the core tokenizer).

> [!Example]- Workflow (Tokenizing Text for Display):
> 1.  **Request:** A component (e.g., the Media Player/Reader) has raw text and its language code.
> ---
> 2.  **Invocation:** The component calls `tokenizerRegistry.tokenizeText(rawText, languageCode)`.
> ---
> 3.  **Selection:** The `Tokenizer Registry` looks up its internal list and finds the highest-priority tokenizer registered for the specified `languageCode` (or falls back to '*'). Let's say it finds a plugin-provided "Chinese Jieba Tokenizer".
> ---
> 4.  **Execution:** The Registry invokes the `tokenize` function associated with the "Chinese Jieba Tokenizer", passing the `rawText`.
> ---
> 5.  **Result:** The tokenizer function processes the text and returns a `Promise<Token[]>`.
> ---
> 6.  **Return:** The Registry resolves the promise and returns the `Token[]` array to the requesting component.
> ---
> 7.  **Usage:** The Media Player/Reader uses the token array (specifically the `startIndex` and `endIndex` for each token) to render the text with interactive elements (e.g., wrapping words in `<span>` tags for dictionary lookups).

> [!Tip] Integration:
> - Receives registrations from [[🧩Faseeh Plugin|BasePlugin]] instances via the `FaseehApp.tokenizers.register` facade.
> - Is utilized by any Renderer component needing to process raw text into interactive or analyzable units (e.g., Player/Reader, potentially search indexing, SRS card generation).

> [!warning] Importance:
> This service is crucial for handling linguistic diversity correctly. A simple whitespace split is insufficient for many languages (e.g., Chinese, Japanese, Thai, German compound words). The pluggable nature allows the community to provide language-specific support.

## API Types

### Registry Interaction Types

> [!Summary]- Registry Interface : `ITokenizerRegistry` (Conceptual)
> ```typescript
> /** Interface defining the public contract for the Text Tokenizer Registry service */
> export interface ITokenizerRegistry {
>   /**
>    * Registers a new Tokenizer function.
>    * @param registration - The complete registration object including metadata and the tokenize function.
>    */
>   register(registration: TokenizerRegistration): void;
>
>   /**
>    * Unregisters a Tokenizer by its unique ID.
>    * @param id - The unique ID of the tokenizer to unregister.
>    */
>   unregister(id: string): void;
>
>   /**
>    * Finds and executes the best available tokenizer for the given text and language.
>    * @param text - The raw text string to tokenize.
>    * @param languageCode - The ISO 639 language code of the text.
>    * @returns A promise resolving to an array of Token objects.
>    */
>   tokenizeText(text: string, languageCode: string): Promise<Token[]>;
>
>   /**
>    * Retrieves metadata about all currently registered tokenizers.
>    * @returns An array of TokenizerInfo objects.
>    */
>   listRegisteredTokenizers(): TokenizerInfo[];
>
>   /**
>    * Retrieves a specific Tokenizer registration by its ID (including the function).
>    * Returns null if not found.
>    * @param id - The unique ID of the desired tokenizer.
>    * @returns The TokenizerRegistration object or null.
>    */
>   getTokenizerById(id: string): TokenizerRegistration | null;
> }
> ```