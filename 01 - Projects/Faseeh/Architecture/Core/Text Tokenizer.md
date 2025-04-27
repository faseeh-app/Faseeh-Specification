---
Type: Abstract Function
---

# Text Tokenizer Function Overview

> [!Note] Description
> Defines the standard contract for providing text tokenization logic to Faseeh. Unlike potentially complex components like OCR engines that might benefit from a base class with lifecycle methods, text tokenizers are often stateless and require minimal setup. Therefore, the core contract is defined by the `TokenizerFunction` signature, which specifies how a function should take raw text and return an array of `Token` objects. Plugins and the core application provide functions matching this signature, along with descriptive metadata (`TokenizerInfo`), to the `Text Tokenizer Registry`.
## Key Notes

> [!Summary]+ Key Aspects:
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Core Contract (`TokenizerFunction`):</span> The essential requirement is to provide an asynchronous or synchronous function that accepts a `text` string and returns (or resolves with) a `Token[]` array, adhering to the defined `Token` interface (`{ text, startIndex, endIndex, isWord }`).
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Statelessness (Typical):</span> Most tokenizer algorithms operate directly on the input text without needing to maintain internal state between calls or perform complex initialization/shutdown. This makes a simple function suitable.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Metadata (`TokenizerInfo`):</span> Alongside the function, essential metadata (unique ID, name, supported languages, priority) must be provided during registration. This allows the `Text Tokenizer Registry` to select the correct function based on context.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Registration:</span> Implementations (the `TokenizerFunction` + `TokenizerInfo`) are registered with the central `Text Tokenizer Registry` service via the `app.tokenizers.register` method provided by the `FaseehApp` API.
> ---
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Execution:</span> The `Text Tokenizer Registry` selects the appropriate registered `TokenizerFunction` based on language/priority and executes it when its `tokenizeText` method is called.

> [!Example]- Interaction & Usage:
> 1.  **Plugin Definition:** A plugin defines a standalone function `myJapaneseTokenizer(text: string): Token[] { /* ... logic using external library ... */ }`.
> ---
> 2.  **Registration (in `onload`):** The plugin calls `this.app.tokenizers.register({ id: 'my-jp-tokenizer', name: '...', languageCodes: ['ja'], tokenize: myJapaneseTokenizer })`.
> ---
> 3.  **Invocation (by Registry):** When needed for Japanese text, the `Text Tokenizer Registry` retrieves `myJapaneseTokenizer` and calls it: `await Promise.resolve(myJapaneseTokenizer(textToProcess))`.
> ---
> 4.  **Unregistration (in `onunload`):** The plugin ideally calls `this.app.tokenizers.unregister('my-jp-tokenizer')`.

> [!Tip] Rationale for Function vs. Class:
> Opting for a `TokenizerFunction` as the primary contract (instead of a `BaseTextTokenizer` class) simplifies the implementation for most common tokenizer use cases. It avoids the overhead of class instantiation and lifecycle management (`initialize`/`shutdown`) when often unnecessary. If a specific tokenizer *did* require complex state or setup, its implementation function could internally manage its own state using closures or module-level variables, while still conforming to the required function signature for registration.

## API Types

### Token Definition

> [!Summary]- Token : `Token`
> Represents a single linguistic unit (typically a word or punctuation) identified by a tokenizer within a larger text. It includes the text itself and its positional information within the original string, crucial for enabling interactive features.
> ```ts
> /** Represents a single token identified in the text */
> export interface Token {
>   text: string;       // The actual token string (e.g., "Hello", ",", "世界")
>   startIndex: number; // Start index in the original raw text
>   endIndex: number;   // End index in the original raw text
>   isWord: boolean;    // Heuristic: is this likely a word vs punctuation/whitespace?
> }
> ```

---
### Tokenizer Definition Types

> [!Summary]- Tokenizer Logic : `TokenizerFunction`
> Defines the required signature for any function that performs text tokenization. It must accept a string and return (or resolve to) an array of `Token` objects.
> ```ts
> /** The function signature for any tokenizer implementation */
> export type TokenizerFunction = (text: string) => Promise<Token[]> | Token[];
> ```

> [!Summary]- Tokenizer Metadata : `TokenizerInfo`
> Describes the identity and capabilities of a tokenizer implementation. This metadata is used by the `Text Tokenizer Registry` to select the appropriate tokenizer based on language and priority.
> ```ts
> /** Metadata describing a Tokenizer's capabilities */
> export interface TokenizerInfo {
>   id: string;                  
>   // Unique ID (e.g., "faseeh-core-whitespace", "plugin-chinese-jieba")
>   name: string;                
>   // Human-readable name (e.g., "Default Whitespace", "Jieba Chinese Tokenizer")
>   languageCodes: string[];     
>   // Array of ISO 639 language codes it supports (e.g., ["*"], ["zh-CN", "zh-TW"], ["ja"])
>   // "*" indicates a fallback/generic tokenizer
>   priority?: number;           
>   // Optional: Higher number means higher priority for the same language code (default 0)
>   description?: string;        
>   // Optional description
> }
> ```

> [!Summary]- Tokenizer Registration : `TokenizerRegistration`
> The complete object provided when registering a tokenizer with the `Text Tokenizer Registry`. It bundles the metadata (`TokenizerInfo`) with the actual implementation logic (`TokenizerFunction`).
> ```ts
> /** The complete registration object provided by core or plugins */
> export interface TokenizerRegistration extends TokenizerInfo {
>   tokenize: TokenizerFunction; // The actual function that performs tokenization
> }
> ```
