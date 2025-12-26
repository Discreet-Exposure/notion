# Telegram AI Notebook Integration for n8n

An intelligent n8n workflow that processes all incoming Telegram messages (links, images, voice notes, videos, documents, and text) and automatically creates organized entries in your Notion "Notebook" database using AI.

## Features

### Supported Content Types
- 🔗 **Links/URLs** - Auto-extracts and processes web links
- 📝 **Text Messages** - Regular text notes and ideas
- 🎤 **Voice Notes** - Auto-transcribed using OpenAI Whisper
- 📷 **Images** - Photos with captions
- 🎬 **Videos** - Video files with descriptions
- 📄 **Documents** - PDFs, files, etc.
- 🎵 **Audio** - Music and audio files

### AI-Powered Processing
The workflow uses GPT-4o to intelligently:

| Field | Description |
|-------|-------------|
| **Title** | Generates a concise, descriptive name for the entry |
| **URL** | Extracts any URLs from the content |
| **Category** | Selects appropriate category (uses existing or creates new) |
| **Tags** | Assigns relevant tags (3-5, uses existing or creates new) |
| **Description** | Generates helpful 2-3 sentence summary |
| **Created Date** | Timestamps when the entry was received |
| **Is Task** | Detects if content is a task/to-do item |
| **Due Date** | Extracts due dates from natural language |
| **Priority** | Assigns High/Medium/Low for tasks |

## Workflow Architecture

```
┌─────────────────┐
│ Telegram Trigger│
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Route Content   │──► Link/Image/Voice/Video/Document/Audio/Text
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Prepare Context │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐  ┌──────────────┐
│ Voice?│  │ Other Content│
└───┬───┘  └──────┬───────┘
    │             │
    ▼             │
┌───────────┐     │
│ Transcribe│     │
│ (Whisper) │     │
└─────┬─────┘     │
      │           │
      └─────┬─────┘
            ▼
┌─────────────────────┐
│ Get Existing Tags/  │
│ Categories from DB  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ AI Content Analyzer │
│ (GPT-4o Agent)      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Parse AI Response   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Create Notion Page  │
└──────────┬──────────┘
           │
      ┌────┴────┐
      ▼         ▼
┌──────────┐ ┌─────────┐
│Has Media?│ │ No Media│
└────┬─────┘ └────┬────┘
     │            │
     ▼            │
┌──────────────┐  │
│ Append Media │  │
│ to Page      │  │
└──────┬───────┘  │
       │          │
       └────┬─────┘
            ▼
┌─────────────────────┐
│ Send Telegram       │
│ Confirmation        │
└─────────────────────┘
```

## Setup Instructions

### Prerequisites
- n8n instance (self-hosted or cloud)
- Telegram account
- Notion account
- OpenAI API key

### Step 1: Create Telegram Bot

1. Open Telegram and search for `@BotFather`
2. Send `/newbot` and follow the prompts
3. Copy the **API Token** you receive
4. Add the bot to your desired chat or use it directly

### Step 2: Set Up Notion Database

Create a Notion database called "Notebook" with these properties:

| Property | Type | Description |
|----------|------|-------------|
| Name | Title | Entry title |
| URL | URL | Link if applicable |
| Category | Select | Content category |
| Tags | Multi-select | Relevant tags |
| Description | Rich Text | AI-generated description |
| Created | Date | When entry was created |
| Is Task | Checkbox | Whether it's a task |
| Due Date | Date | Task due date |
| Priority | Select | High/Medium/Low |
| Content Type | Select | Link/Image/Voice/etc. |
| Source | Rich Text | Always "Telegram" |

**Recommended Categories to pre-create:**
- Articles
- Resources
- Ideas
- Tasks
- Notes
- Projects
- References
- Learning
- Entertainment
- Work
- Personal

### Step 3: Create Notion Integration

1. Go to [Notion Developers](https://www.notion.so/my-integrations)
2. Click "New Integration"
3. Name it "Telegram Notebook Bot"
4. Copy the **Internal Integration Token**
5. Go to your Notebook database in Notion
6. Click `...` → "Add connections" → Select your integration

### Step 4: Get Notion Database ID

1. Open your Notebook database in Notion
2. Copy the URL - it looks like:
   ```
   https://notion.so/workspace/DATABASE_ID?v=...
   ```
3. The DATABASE_ID is the 32-character string before the `?`

### Step 5: Import Workflow to n8n

1. Open n8n
2. Go to Workflows → Import from File
3. Select `telegram-notebook-workflow.json`
4. The workflow will be imported

### Step 6: Configure Credentials

In n8n, set up these credentials:

#### Telegram API
- Name: `Telegram Bot API`
- Access Token: Your bot token from BotFather

#### Notion API
- Name: `Notion API`
- API Key: Your integration token

#### OpenAI API
- Name: `OpenAI API`
- API Key: Your OpenAI API key

### Step 7: Update Database IDs

In the workflow, update these nodes with your Notion Database ID:
- `Get Notebook Schema`
- `Get Existing Tags/Categories`
- `Create Notebook Entry`

Replace `YOUR_NOTEBOOK_DATABASE_ID` with your actual database ID.

### Step 8: Activate Workflow

1. Click "Save"
2. Toggle the workflow to "Active"
3. Your bot is now ready!

## Usage Examples

### Saving a Link
```
Send: https://example.com/interesting-article
Bot Response: ✅ Saved to Notebook!
📝 Interesting Article About Topic
📂 Category: Articles
🏷️ Tags: reading, technology, reference
💡 This article discusses...
```

### Creating a Task
```
Send: Remember to review the project proposal by Friday
Bot Response: ✅ Saved to Notebook!
📝 Review Project Proposal
📂 Category: Tasks
🏷️ Tags: work, review, deadline
✔️ Marked as Task
📅 Due: 2024-12-27
💡 Task to review the project proposal before the end of the week.
```

### Voice Note
```
Send: [Voice message about meeting notes]
Bot Response: ✅ Saved to Notebook!
📝 Meeting Notes - Q4 Planning
📂 Category: Notes
🏷️ Tags: meeting, planning, Q4
💡 Discussion points from the Q4 planning meeting including...
```

### Quick Idea
```
Send: New app idea: AI-powered recipe generator based on fridge contents
Bot Response: ✅ Saved to Notebook!
📝 AI Recipe Generator App Idea
📂 Category: Ideas
🏷️ Tags: app-idea, AI, cooking, startup
💡 Concept for an application that uses AI to suggest recipes based on available ingredients.
```

## Customization

### Modify Categories
Edit the AI agent's system prompt to include your preferred categories:
```
Common categories: Articles, Resources, Ideas, Tasks, Notes, Projects, References, Learning, Entertainment, Work, Personal, [Your Categories]
```

### Adjust AI Behavior
Modify the `AI Content Analyzer` node's prompt to:
- Change how titles are generated
- Adjust tag limits
- Modify description length
- Add custom fields

### Add More Content Types
The router supports additional Telegram content types:
- Location
- Contact
- Sticker
- Poll

## Troubleshooting

### Bot Not Responding
- Check webhook is set up correctly
- Verify bot token is valid
- Ensure workflow is active

### Notion Errors
- Verify integration has access to database
- Check database ID is correct
- Confirm all properties exist with correct types

### Voice Transcription Failing
- Check OpenAI API key is valid
- Verify Whisper API access
- Check voice file size limits

### AI Not Categorizing Correctly
- Add more context to prompts
- Pre-create common categories/tags in Notion
- Adjust temperature setting for more consistent output

## Cost Considerations

This workflow uses:
- **OpenAI GPT-4o**: ~$0.01-0.03 per entry
- **OpenAI Whisper**: ~$0.006 per minute of audio
- **Notion API**: Free
- **Telegram API**: Free

Estimated cost: ~$3-10/month for moderate usage (100-300 entries)

## License

MIT License - Feel free to modify and use as needed.

## Contributing

Suggestions and improvements welcome! Common enhancements:
- Add support for forwarded messages
- Implement duplicate detection
- Add weekly digest summaries
- Support multiple Notion databases
