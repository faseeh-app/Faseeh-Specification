#LOGIC

---
This Parser is an essential component to many features, its role is to extract language units (words, sentences, idioms, ...).
- Once a text is imported (article, caption, etc.), the parser will analyze it and extract all words.
- Since the parsing process might be language dependent, we will have to modularize the parsing process.
	- Using different parsers for different languages.
	- Allowing the community to create their own parsers.
	- Allowing the user to select the parser he wants to use.

