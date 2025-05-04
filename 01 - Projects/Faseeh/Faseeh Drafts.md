# Faseeh

## Description
Faseeh is a cross-platform language learning toolkit that aims to provide a comprehensive set of tools to help users learn a new language by transferring their favorite youtube videos, podcasts, articles and more into interactive language learning materials, 


### Goals
- Overcome the problem of boring language learning materials.
- Centralize the all the language learning tools in one app with small to no setup.
- Provide a free alternative to existing paid solutions.

description 2: 
Ai toolkit platform that let the user to transform the content he enjoys from the web into interactive language learning materials, it comes as a solution to the rarity of imerssive language learning materials that are both engaging and easy to setup.
- **Media Import**: users can import different types of media into the app to build their own language learning materials.
- **Subtitles Generation**: users can generate high-quality subtitles for their imported media.
- **Integrated Dictionary**: users can hover over any word to get its definition from an external dictionary. (dictionaries are replaceable by plugins)
- **Vocabulary extraction**: articles, subtitles and any text based media supports vocabulary/sentence extraction with advanced customization options :
- **Flashcards**: users can create flashcards from extracted vocabulary and sentences.
- the platform support community plugins to extend its functionality.
and more... 
## <span style="color:rgb(255, 0, 0)">Features</span>

- **Media Import**: users can import different types of media into the app to build their own language learning materials.
- **Subtitles Generation**: users can generate high-quality subtitles for their imported media.
- **Vocabulary extraction**: articles, subtitles and any text based media supports vocabulary/sentence extraction with advanced customization options :
    - pronounciation
    - image
    - translation
    - custom definition
    - dictionary's definiton
    - proficiency level
- **Flashcards**: users can create flashcards from extracted vocabulary and sentences.
- **Integrated Dictionary**: users can hover over any word to get its definition from an external dictionary. (dictionaries are replaceable by plugins)
- **advanced media player controls**: add bookmarks; rewind or fast forward by time, sentence, bookmarks;
- **Pronouciation checker**: users can record their pronounciation, analyze it and compare it with original media.
- **Progress tracking**: users can track their progress and see their learning statistics.

#### Pronouciation Checker #LOGIC 
A tool to help users improve their pronouciation by analyzing their recordings and reporting the results to them.
- Should support as many languages and accents as possible.
- Should be as accurate as possible by using well known models and algorithms.
- We can change the approach based on the language and the accent.
- The pronounciation is analyzed against either the original media, a reference audio or just the text's phonetics.
- Feedback may vary based on the used approach.

#### Media Importing #LOGIC
User can import media from different sources and formats such as local videofiles, youtube videos, streamable videos, podcasts, articles, web pages, epubs, pdf (if possible), etc.
- <span style="color:rgb(146, 208, 80)">**Manual import :**</span>
	- user can import any type of media from their <span style="color:rgb(0, 176, 240)">local machine, or clipboard</span>.`(local files, urls, copied text, ...)`
	- this option serves better for content that is <span style="color:rgb(0, 176, 240)">not available online or copyrighted content</span>.
- <span style="color:rgb(146, 208, 80)">**Automatic import :**</span>
	- browser <span style="color:rgb(0, 176, 240)">**extension**</span> that allows users to import media from the web with a <span style="color:rgb(0, 176, 240)">**single click**</span>.
	- <span style="color:rgb(0, 176, 240)">**Convenient**</span> for most of the content available online<sub> *(Checks later for DRM protected content).*</sub>
	- Requires a way to <span style="color:rgb(0, 176, 240)">extract the main content from the page</span> the user is browsing/saving, for example in youtube this would mean the video url and its metadata, in a blog this would mean the article content and its metadata.

#### Subtitle Generation #LOGIC
A solution to the lack of video content with accurate subtitles in the source/target language. by using the latest AI models to generate high-quality subtitles for the imported media.
- <span style="color:rgb(146, 208, 80)">**Online Option *(Paid)* :**</span>
	- We might provide a <span style="color:rgb(0, 176, 240)">paid service from the backend</span> to generate subtitles for the imported media.
	- or we can provide an <span style="color:rgb(0, 176, 240)">integration with an existing paid service</span>, so that the user can use it directly from the app by providing an api key for example.
	- Provide integration with <span style="color:rgb(0, 176, 240)">online subtitle databases</span> such as opensubtitles.org, subs.com, etc. 
- <span style="color:rgb(146, 208, 80)">**Offline Option *(Free)* :**</span>
	- We can provide an offline option to <span style="color:rgb(0, 176, 240)">download and setup the models locally</span> on the user's machine.
	- the user can also <span style="color:rgb(0, 176, 240)">import subtitle files manually</span>.


#### Mouse-over Dictionary #UI
Integrated dictionary that allows users to hover over any word in the imported media to get its definition.
- The dictionary sources should be <span style="color:rgb(0, 176, 240)">replaceable</span> to cover different languages, dialects and the preferences of the user.
- Dictionaries sources can be either <span style="color:rgb(146, 208, 80)">online</span> or <span style="color:rgb(255, 0, 0)">offline</span>.
- The layout of the dictionary should be customizable by the user, to fit their needs. for example:
	- `definition only.`
	- `definition and the image.`
	- `definition and the translation.`
	- `definition and the pronunciation.`
	- `definition and the example sentence.`
	- `definition and the synonyms.`
	- `definition and the antonyms.`

#### Advanced Media Player
Media player that is adpted for language learning purposes.
- Dual subtitles support.
- Mouse-over dictionary.
- Word/Sentence highlighting.
- Seeking bar with bookmarks.
- Rewind/Fast forward by time, sentence, bookmarks.
- Sentence/Word repeat.
- Clip extraction for selected sentences/words. <span style="color:rgb(184, 184, 184)">*(to be used with other tools)*</span>

### <span style="color:rgb(255, 192, 0)">⚠️</span> <span style="color:rgb(184, 184, 184)">**Notes**</span>

-  Consider downloading video/audio files locally for offline use as a feature.

## Architecture

### Plugin System
Notes on how the plugin system will work:
- Plugins can be used to add new features, replace existing features or extend existing features.
- Plugins can communicate through an event system.
- Plugins depend on the core system and other plugins.

### Tasks
- build the core system
    - event system : either we can use an existing event system or build our own.
    - plugin system :
        
        - define the plugin system foundation
            - manifest file that contains metadata about the plugin such as name, version, author, description, dependencies, entry point, etc.
            - plugin loader that loads the plugin and registers it with the core system.
            - plugin api: a set of interfaces/methods that plugins can use to interact with the core system to change its behavior or the ui.
            - plugin manager: a manager that manages the lifecycle of plugins.
        
        - media importer:
            - user can import video manually and text to the app to get access to the language learning tools.
            - media importer should be also implemented as a browser extension and a logic layer functionality that interact together to extract the main content from the page the user is browsing/saving, for example in youtube this would main the video url and its metadata, in a blog this would mean the article content and its metadata.
            - Note: this one could either be a core functionality or a default plugin that comes with the app.
            - Note: We need a general way to extract the main content from any page, otherwise we have to let the community to develop website dedicated media importers.

        
        - Media player:
            - we can either use an existing media player or build our own as long as we meet our customization requirement.
            - we need a programming control over the media player to be able to :
                - add overlays on top of the video.
                - control the rewind and fast forward functionality.
                - The seeking bar so that we can add bookmarks or change color.

        - Local database: to save generated content and user progress.

- Default Plugins : (these are plugins that comes bundled with app but can be desactivated or replaced by community ones)
    - subtitle generator:
        - single click ai model downloader
        - generate subtitles for the video
        - generate translation
        - batch subtitle geration for playlists 



**How the Demo video should be:** we start by openning the app which will show the feed page it contains all the content the user has imported into the app, we will import a new media manually, then import one from the web, then show from a video with pre generated subtitle how we can watch it inside the app. after this in the settings window we activate the subtitle generating plugin, and we go bach to one of the video to show the appear of new button we click on one of them and show the results.



## User Interface
- Feed page: contains all the content the user has imported into the app. (Card for each media)
- 


# Brief Description


Faseeh est une application d’apprentissage des langues qui propose une boîte à outils combinant des technologies d’IA et des outils classiques pour aider les utilisateurs à apprendre une nouvelle langue. Elle transforme les contenus en ligne que vous appréciez en supports d'apprentissage immersifs et interactifs.

Faseeh est également une plateforme communautaire où les utilisateurs peuvent développer et partager leurs propres plugins pour étendre les fonctionnalités de l'application. Ils peuvent aussi partager avec la communauté les supports d’apprentissage qu’ils ont créés.

l'objectif de Faseeh est de rendre l'apprentissage des langues plus engageant, et au même temps augmenter la variété, la quantité et la facilité d'accès aux supports d'apprentissage "immersifs".