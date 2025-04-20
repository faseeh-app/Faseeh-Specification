# Faseeh : Language Learning Toolkit App

## Overview
Faseeh is a language learning platform designed to redefine traditional methodologies by prioritizing flexibility, personalization, and learner autonomy. Unlike conventional platforms that rely on structured grammar lessons or fixed content libraries, Faseeh empowers users to learn languages through material they find inherently engaging—whether books, films, podcasts, social media, or other digital content. Inspired by techniques employed by polyglots and language enthusiasts, the platform transforms everyday content into dynamic learning opportunities.

### Core Features
Faseeh has unique <span style="font-weight:bold; color:rgb(0, 176, 240)">core features</span> that set it apart from other language learning platforms :
- <span style="font-weight:bold; color:rgb(146, 208, 80)">Content-Agnostic Flexibility : </span>Faseeh is not confined to predefined lessons or materials. Instead, it supports learning through virtually any digital or text-based content, enabling users to engage with languages in contexts that align with their personal interests. This approach ensures relevance and sustains motivation by integrating learning into activities users already enjoy.
- <span style="font-weight:bold; color:rgb(146, 208, 80)">Plugin Ecosystem for Community Innovation :</span> Faseeh’s open architecture allows users and developers to create and share plugins, expanding the platform’s capabilities. Community-developed plugins can introduce new tools (e.g., specialized grammar analyzers, dialect-specific resources) or integrate support for underrepresented languages. This extensibility ensures the platform evolves alongside learner needs and global linguistic diversity.
- <span style="font-weight:bold; color:rgb(146, 208, 80)">Collaborative Community Ecosystem : </span>The platform fosters a global community where users collaboratively refine tools, share learning strategies, and crowdsource solutions. Plugins, tutorials, and shared content are curated in a public repository, democratizing access to high-quality resources regardless of technical expertise or financial capacity.
- <span style="font-weight:bold; color:rgb(146, 208, 80)">Integrated Multifunctional Toolkit : </span>Faseeh gathers tools and ideas from various language learning techniques and combines them into puts them in one place. This makes it easy for users to access everything they need to learn a new language without having to switch between different apps or platforms.
### Philosophy & Vision
Faseeh operates on the principle that language acquisition thrives when aligned with personal interests and real-world contexts. Rather than isolating learning to rigid exercises, the platform bridges the gap between education and daily life, by transformting passive content consumption into an active, immersive learning experience, making fluency a natural outcome of curiosity-driven exploration. 

Here’s a structured breakdown of **features**, their functionalities, use cases, and related technical details:
## Features
### **1. Media Importing**  
Allows users to import content from diverse sources (local files, URLs, streaming platforms, etc.) into Faseeh for language learning.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">How It Works</span>
- **Manual Import**: Users drag-and-drop files (videos, PDFs, EPUBs) or paste URLs/copied text.  
- **Automatic Import**: Browser extension detects media (e.g., YouTube videos, blogs) and imports metadata (title, transcript, URL).  
- **Content Extraction**: Strips non-essential page elements (ads, menus) to isolate primary content (e.g., article text, video links).  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Usage Scenarios</span>  
- A user imports a French cooking blog to study food-related vocabulary.  
- A learner adds a German podcast episode via URL for listening practice.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Technical Details & Concerns</span>  
- **DRM Handling**: Avoids copyrighted/DRM-protected content; warns users during import.  
- **Format Support**: Libraries like `youtube-dl` for video extraction, `pdf.js` for PDF parsing.  
- **Performance**: Optimized for large files (e.g., 4K videos) via chunked processing.  

---

### **2. Subtitle Generation**  
Generates or integrates subtitles for videos to support comprehension and vocabulary acquisition.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">How It Works</span> 
- **Online (Premium)**: Uses cloud-based AI models (e.g., OpenAI Whisper) for real-time subtitle generation. Users provide API keys for third-party services.  
- **Offline (Free)**: Locally runs lightweight models (e.g., Vosk) for subtitle creation.  
- **Subtitle Databases**: Pulls existing subtitles from OpenSubtitles.org.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Usage Scenarios</span>  
- A learner generates Japanese subtitles for an anime episode to study casual speech.  
- A user uploads a manually corrected `.srt` file for a documentary.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Technical Details & Concerns</span>  
- **Model Accuracy**: Benchmarking AI models for low-resource languages (e.g., Swahili).  
- **Synchronization**: Aligning subtitles with audio using FFmpeg.  
- **Storage**: Caching subtitles in JSON format with timestamps and confidence scores.  

---

### **3. Language Units Parser**  
Extracts words, phrases, and idioms from imported text for vocabulary tracking.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">How It Works</span> 
- **Modular Parsers**: Language-specific NLP pipelines (e.g., spaCy for English, SudachiPy for Japanese).  
- **Community Plugins**: Users create parsers for dialects or niche languages (e.g., Basque).  
- **Customization**: Users select parsers for specific needs (e.g., medical terminology).  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Usage Scenarios</span>  
- Parsing a Spanish novel to extract idioms for flashcards.  
- A developer contributes a parser for Cantonese compound words.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Technical Details & Concerns</span>  
- **Tokenization**: Handling languages without spaces (e.g., Chinese).  
- **Performance**: Optimizing for large texts (e.g., 500-page EPUBs).  
- **Plugin API**: Standardized input/output formats for community parsers.  

---

### **4. Advanced Media Player-Reader**  
A unified interface for interacting with videos, podcasts, and text with language-learning tools.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">How It Works</span> 
- **Dual Subtitles**: Displays source/target languages simultaneously.  
- **Interactive Tools**:  
  - Hover dictionary with definitions, images, and examples.  
  - Clip extraction to save challenging audio snippets.  
  - Playback speed control (0.5x–2x).  
- **Bookmarks**: Tag key moments (e.g., grammar explanations).  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Usage Scenarios</span>  
- Slowing down a Portuguese news clip to practice listening.  
- Extracting a Mandarin dialogue clip for repetition.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Technical Details & Concerns</span>  
- **Latency**: Minimizing lag in subtitle rendering with WebGL.  
- **Cross-Platform**: Using React Native for mobile/desktop sync.  
- **Accessibility**: Screen reader support for text content.  

---

### **5. Mouse-Over Dictionary**  
Provides instant definitions, translations, and context for words in media.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">How It Works</span> 
- **Dynamic Sources**: Pulls from offline dictionaries (e.g., StarDict) or APIs (Google Translate).  
- **Custom Layouts**: Users choose display options (e.g., “Definition + Image”).  
- **User Notes**: Add personal mnemonics or example sentences.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Usage Scenarios</span>  
- Hovering over “amigo” in a Spanish film to see usage examples.  
- A user adds a custom note: “*Bonjour* = formal greeting.”  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Technical Details & Concerns</span>  
- **API Rate Limits**: Caching frequent queries to avoid overages.  
- **Offline Support**: Bundling dictionaries for languages like Icelandic.  
- **UI Responsiveness**: Debouncing hover events to reduce flicker.  

---

### **6. Pronunciation Checker**  
Analyzes user-recorded speech against native references for feedback.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">How It Works</span> 
- **Model Integration**: Uses Mozilla DeepSpeech or OpenAI Whisper for speech-to-text.  
- **Accent Adaptation**: Community-trained models for regional accents (e.g., Southern U.S. English).  
- **Feedback Types**: Visual waveforms, phonetic alignment scores.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Usage Scenarios</span>  
- Practicing French vowel sounds with real-time feedback.  
- Comparing pronunciation to a podcast host’s speech.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Technical Details & Concerns</span>  
- **Latency**: Optimizing model inference time for real-time use.  
- **Data Privacy**: Local processing for sensitive recordings.  
- **Accuracy**: Handling homophones (e.g., “there” vs. “their”).  

---

### **7. Spaced Repetition System (SRS)**  
Optimizes review schedules for vocabulary retention.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">How It Works</span> 
- **Algorithm**: Adjusts intervals using SM-2 or FSRS algorithms.  
- **Familiarity Tracking**: Auto-tags words as “New” → “Familiar” → “Mastered.”  
- **Media Integration**: Words encountered in videos/articles are added to SRS queues.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Usage Scenarios</span>  
- Reviewing medical German terms before an exam.  
- Prioritizing words from a user’s favorite TV show.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Technical Details & Concerns</span>  
- **Data Syncing**: Conflict resolution for offline/online edits.  
- **Customization**: Letting users tweak algorithm parameters.  
- **Scalability**: Handling 10,000+ cards per user.  

---

### **8. Content Organization**  
Hierarchical system to manage media, groups, and collections.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">How It Works</span> 
- **Media Objects**: Individual items (e.g., a TED Talk video).  
- **Groups**: Auto-generated series (e.g., podcast episodes).  
- **Collections**: User-curated playlists (e.g., “Business Japanese”).  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Usage Scenarios</span>  
- Grouping all “Harry Potter” audiobooks for sequential learning.  
- Creating a “Travel Spanish” collection with phrases and videos.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Technical Details & Concerns</span>  
- **Database Schema**: GraphQL for nested groups/collections.  
- **Search**: ElasticSearch for fast metadata queries (e.g., “find all B1-level French videos”).  
- **Backup**: Syncing to cloud storage (e.g., Dropbox).  

---

### **9. Metadata & Extensibility**  
Flexible metadata system for custom tagging and plugin integration.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">How It Works</span> 
- **Static Fields**: Language, creation date, media type.  
- **Dynamic Fields**: Custom JSON data (e.g., “difficulty_score”: 7).  
- **Plugin API**: Developers inject metadata (e.g., “grammar_rules”: [“past_tense”]).  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Usage Scenarios</span>  
- A plugin adds “cultural_notes” metadata for historical videos.  
- Tagging media as “A1” or “C2” for filtered practice.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Technical Details & Concerns</span>  
- **Validation**: Schema enforcement for critical fields.  
- **Performance**: Indexing dynamic fields for fast queries.  
- **Versioning**: Handling changes to plugin-generated metadata.  

---

### **10. Community-Driven Plugins**  
Allows users/developers to extend Faseeh’s functionality.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">How It Works</span> 
- **Plugin Marketplace**: Share tools (e.g., a Cantonese parser).  
- **API Hooks**: Integrate with third-party services (e.g., Anki sync).  
- **Resource Sharing**: Distribute community-generated subtitles/vocab lists.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Usage Scenarios</span>  
- A developer creates a plugin for analyzing Arabic poetry.  
- Users share annotated subtitles for a niche documentary.  

<span style="font-weight:bold; color:rgb(146, 208, 80)">Technical Details & Concerns</span>  
- **Security**: Sandboxing plugins to prevent malware.  
- **Compatibility**: Versioning APIs to avoid breaking changes.  
- **Monetization**: Allowing paid plugins with revenue sharing.  

## Architecture

### Considerations
- The app should be an offline first application that can sync with the cloud.
- User data should be kept stored locally as much as possible.
- Features/services that are language dependent should be implemented as plugins. therefore they should be kept in the client side.
- Heavy processing features should be kept in the server side.
### **1. Client Side**
- Separate UI components and Data, so that we can let the user define new UI components.
- Plugins should depends on core features interfaces not their implementations.
- 
#### Core Features
- Media Importer :
	- Extension should only send the url & html content of the page to the app.
	- The extension imported content should have website dedicated parser.
	- The manual imported content should have format dedicated parser.
- Content Parser :
	- Content parser should has a minimum number of implementations by default, and the community can add new implementations.
	- The content parser should be able to parse the content in a language agnostic way.
- Text Tokenizer :
	- The text tokenizer should have one fallback implementation that can be used for any language and then there will be language specific implementations.
- Storage Service :
	- Storage Service is the interface between the app, the database and the file system.
	- The Storage Service implementation defines how each file format is stored and where.
	- It's better to have a config file that defines all paths and let the service to expose the paths to the app and the plugins.
-