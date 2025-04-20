# Overview
Content Adapters handle the conversion, standardization, and interpretation of different file formats so they can be seamlessly integrated into Faseeh. They act as "translators" between raw files `(EPUBs, videos, PDFs, website page...)` and the structured data Faseeh needs to work with, in other words to [[MediaObject]]. Think of them as middleware that ensures _any file type_ can be stored, parsed, and used by Faseeh or plugins, regardless of its original format.

## Key Notes

> [!Note] Responsibilities
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Main Content Extraction</span>, extract textual content that can be processed by Faseeh
> ```
> Video/Audio --> Subtitles
> Html --> Main content text
> ```
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Metadata Extraction</span>, extract metadata from the file
> ```
> EPUB/PDF --> Title, Author, Cover, etc
> Video --> Title, Duration, Thumbnail, etc
> ```
> - <span style="font-weight:bold; color:rgb(146, 208, 80)">Standardization</span>, convert the the extracted content and metadata into [[MediaObject]] format, so that Faseeh and plugins can work with it.


```ts
interface ContentAdapter {
  // Check if the adapter supports a file type, using library, magic numbers, or file extension (not recomended)
  supports(file: File | string): boolean;

  // Convert the file into a standardized format
  process(input: File | string): Promise<MediaObject>;
}
```

```ts
class EpubAdapter implements FileFormatAdapter {
  supports(file: File) {
    return file.name.endsWith(".epub");
  }

  async process(file: File) {
    const epub = await parseEpub(file); // library: epub.js
    return {
      id: generateUUID(),
      type: "book",
      title: epub.metadata.title,
      content: epub.text,
      language: detectLanguage(epub.text),
    };
  }
}
```