# Sync Security+ Notes → Anki

Read all markdown files recursively inside the `Security+` folder in the current vault directory.

For each file, extract content that should become flashcards — definitions, key concepts, acronyms, numbered lists of facts, Q&A patterns, etc. Security+ specific: protocols, port numbers, attack types, cryptography concepts, compliance frameworks, tools.

Then:

1. Connect to AnkiConnect at http://localhost:8765
2. Fetch all existing cards from the deck "Security+" (create it if it doesn't exist)
3. Compare new candidate cards against existing ones — skip any where the front already exists (case-insensitive match)
4. Add only net-new cards using the Basic note type

Card generation rules:

- One concept per card
- Front: a clear question or prompt (e.g. "What port does HTTPS use?", "What is a Man-in-the-Middle attack?")
- Back: concise answer with just enough context
- For acronyms: Front = acronym, Back = full name + one-line description
- Do NOT create cards for headings, wikilinks, MOC structure, or navigation bullets
- Skip any file named `_index`, `MOC`, or `README`

After finishing, print a summary:

- Files scanned
- Candidate cards found
- Skipped (already in Anki)
- Added
