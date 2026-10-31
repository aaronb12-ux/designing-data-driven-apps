
# Full-Text Search

**Full-text search** allows you to search a collection of text documents for keywords that might appear anywhere in the text.

Full-text search often goes beyond exact keyword matching. It can also involve matching similar words, different word forms, and synonyms.

At its core, we can think of full-text search as another type of multidimensional query:

- Each word that might appear in a document represents a dimension.
- A document that contains a particular word has a value of `1` in that dimension.
- A document that does not contain that word has a value of `0`.

The data structure that many search engines use to answer these queries is called an **inverted index**.

An inverted index is a key-value structure where:
- **Key:** A term or word.
- **Value:** A list of document IDs for all documents that contain that term.

This allows a search engine to quickly find documents containing a particular word without scanning every document.
