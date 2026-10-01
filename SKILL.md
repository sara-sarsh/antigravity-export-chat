---
name: export-chat
description: Exports conversations to readable Markdown or creates a complete migration/import package for another Antigravity instance. Activate when the user asks to export chat history, package project chats for a colleague/friend, or share conversation history.
---

# Export Chat Skill

Depending on the user's intent, execute either **Mode A (Single Chat Export)** or **Mode B (Full Project Migration & Packaging)**:

---

## Mode A: Single Conversation Export (Markdown Only)

Use when the user asks to export or save the current conversation to Markdown.

1. **Locate the Transcript:**
   The conversation history is stored in your brain directory:
   `<appDataDir>/brain/<conversation-id>/.system_generated/logs/transcript.jsonl`
   (or `transcript_full.jsonl` if detailed content was compacted).

2. **Read & Parse the Transcript:**
   - Parse each JSON line in `transcript.jsonl`.
   - Extract messages from the `USER` (type: `USER_INPUT`). Clean out `<USER_REQUEST>` and internal tags.
   - Extract visible responses from the `MODEL`.
   - Wrap verbosity, thoughts, and tool action summaries in **collapsible HTML blocks**:
     `<details><summary>🧠 مشاهده روند فکری و ابزارها</summary>...</details>`
   - If the conversation is in Persian/RTL, ensure the entire document is wrapped inside `<div dir="rtl" style="text-align: right;">` and `</div>`.

3. **Save to File:**
   - Write to the workspace (`chat_export.md` or a user-specified name).
   - Provide a clickable markdown link: `[chat_export.md](file:///path/to/chat_export.md)`.

---

## Mode B: Full Project Migration & Sharing Package (Antigravity Import + Offline MD)

Use when the user asks to share all project chats with a teammate/friend, export history to another machine, or create an importable archive for Google Antigravity.

1. **Identify All Project Conversations:**
   - Query `<appDataDir>/conversation_summaries.db`:
     ```python
     import sqlite3
     conn = sqlite3.connect('<appDataDir>/conversation_summaries.db')
     cur = conn.cursor()
     # Match by workspace URI or title
     cur.execute("SELECT conversation_id, title, preview, workspace_uris FROM conversation_summaries WHERE workspace_uris LIKE ? OR title LIKE ?", ('%' + project_name + '%', '%' + project_name + '%'))
     ```
   - Alternatively, search transcripts in `<appDataDir>/brain/**/transcript.jsonl` for matches to the project workspace directory path.

2. **Structure the Export Directory:**
   ```text
   <project_name>_chat_export/
   ├── import_to_antigravity.py      # Automated cross-platform import script
   ├── README.md                     # Bilingual/Persian guide with manual & automated steps
   ├── readable_markdown/            # Clean, formatted Markdown versions for offline reading
   │   ├── 01_phase_one.md
   │   ├── 02_phase_two.md
   │   └── artifacts/                # Exported plans and walkthroughs (*.md)
   └── antigravity_raw_data/         # Raw system data for Antigravity App/IDE
       ├── conversations/            # SQLite databases (<id>.db)
       ├── brain/                    # Complete brain directories (<id>/)
       └── metadata.json             # Exported metadata from conversation_summaries.db
   ```

3. **Data Consolidation Rules:**
   - **SQLite Databases:** Use the `sqlite3` backup API to produce a clean, self-contained `<id>.db` file that merges any pending WAL (`-wal`) and shared memory (`-shm`) transactions.
   - **Brain Folders:** Copy `<appDataDir>/brain/<id>` including `.system_generated/logs/`, `tasks/`, and `messages/`.
   - **Artifacts:** Copy any `implementation_plan.md` or `walkthrough.md` files from brain folders into `readable_markdown/artifacts/`.

4. **Provide `import_to_antigravity.py`:**
   Generate a standalone, zero-dependency Python 3 script that:
   - Identifies the operating system (Windows: `%USERPROFILE%\.gemini\antigravity`, macOS/Linux: `~/.gemini/antigravity`).
   - Copies `conversations/*.db` and `brain/<id>` folders into destination directories.
   - Inserts or updates records into destination `conversation_summaries.db` so the conversations immediately appear in Antigravity's UI sidebar.

5. **Compress & Present:**
   - Package the export directory into `<project_name>_chat_history.zip`.
   - Present clickable links to both the ZIP and the exported folder.
