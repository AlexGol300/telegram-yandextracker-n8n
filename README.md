# Tracker Telegram Bot for n8n

An n8n automation workflow that connects a Telegram bot to Yandex Tracker.
Users create and update tasks via natural language messages in Telegram. AI parsing extracts task content and deadlines automatically.

## Overview

The workflow connects Telegram, n8n, OpenAI, and Yandex Tracker into one task management flow.
The bot accepts commands, maintains per-user dialog context between messages, creates issues in Yandex Tracker, and stores synchronization data in local Data Tables.



## Use Case

Instead of opening Yandex Tracker manually, the user sends a message in Telegram such as `/newtask call the client, deadline tomorrow at 10:00`. The workflow parses the text, creates the issue in Yandex Tracker, and saves the task metadata locally in `Task_Table`.

## Architecture

| Component | Tool |
|---|---|
| Automation engine | n8n |
| Messenger | Telegram Bot API |
| Task tracker | Yandex Tracker API |
| AI parsing | OpenAI (GPT) |
| Data storage | n8n Data Tables |

The workflow processes each message through the following pipeline:

1. **Telegram Trigger** — receives the incoming message.
2. **Command parser** — detects `/start`, `/newtask`, `/updatetask`, `/list`, `/cancel`.
3. **Context storage** — reads and writes current user dialog state from `context_command_table`.
4. **AI parser** — extracts structured task content and deadlines from free-form text.
5. **Yandex Tracker integration** — creates or updates issues via Yandex Tracker API.
6. **Task storage** — saves and updates task metadata in `Task_Table`.

## Data Model

### Task_Table

Stores all tasks created through the bot, synchronized with Yandex Tracker.



| Field | Type | Description |
|---|---|---|
| `chatid` | number | Telegram chat ID of the user |
| `TaskContent` | string | Task description |
| `DeadlineAt` | dateTime | Task deadline in ISO 8601 |
| `Finished` | boolean | Whether the task is marked as finished |
| `notified` | boolean | Whether the user has been notified |
| `Tracker_Task_ID` | string | Yandex Tracker internal object ID (used in API calls) |
| `Tracker_Key` | string | Human-readable issue key, e.g. `RAZVITIE-15` (shown to user only) |
| `LastSyncStatus` | string | Last sync result: `created`, `finished` |

### context_command_table

Stores the current dialog state for each Telegram user. Allows multi-step interactions across separate messages.



| Field | Type | Description |
|---|---|---|
| `chatid` | number | Telegram chat ID of the user |
| `command` | string | Active command, e.g. `updatetask` |
| `step` | string | Current step in the dialog, e.g. `wait_updatetask_payload` |
| `draft_taskcontent` | string | Temporary task content draft |
| `draft_deadlineat` | string | Temporary deadline draft |
| `draft_taskid` | string | Task ID being edited |
| `draft_action` | string | Pending action: `finish`, `changdeadline`, `remove` |
| `updated_at` | dateTime | Last update to this context record |

## Task Identifiers

Each task has three identifiers — it is important not to confuse them:

| ID | Field | Example | Usage |
|---|---|---|---|
| Local DB row ID | `id` | `1` | Internal n8n table reference |
| Yandex Tracker object ID | `Tracker_Task_ID` | `6a33d392ed24231007ea34ea` | Used in API calls to Yandex Tracker |
| Human-readable issue key | `Tracker_Key` | `RAZVITIE-15` | Shown to users in Telegram messages |

## Yandex Tracker Queue



## Commands

| Command | Description |
|---|---|
| `/start` | Shows welcome message |
| `/newtask` | Starts task creation flow |
| `/updatetask` | Starts task update flow |
| `/list` | Shows active tasks |
| `/cancel` | Cancels current operation and clears dialog context |

## Workflow Behavior

### Create Task (`/newtask`)

1. User sends `/newtask` with task description.
2. AI parses task text and deadline.
3. Issue is created in Yandex Tracker.
4. Task metadata is saved to `Task_Table`.
5. Bot replies with the Tracker key and deadline confirmation.

### Update Task (`/updatetask`)

1. User sends `/updatetask`.
2. Bot asks for the task to update (by number from `/list`).
3. User selects an action: finish, change deadline, or remove.
4. Changes are applied in Yandex Tracker.
5. `Task_Table` is updated with the new sync status.

### Cancel (`/cancel`)

The bot deletes the active row from `context_command_table` and confirms cancellation.
Yandex Tracker and `Task_Table` are not affected.

## Deadline Format

Deadlines must be in ISO 8601 format with Moscow time (UTC+3):

```
2026-06-20T10:00:00.000+03:00
```

The AI node converts user input (e.g. "tomorrow at 10:00") into this format automatically.
Yandex Tracker rejects any other date format.

## Requirements

- n8n instance (self-hosted or cloud)
- Telegram bot token
- Yandex Tracker OAuth token and organization ID
- OpenAI API key
- Two configured n8n Data Tables: `Task_Table` and `context_command_table`

## Setup

1. Create credentials in n8n: Telegram API, Yandex Tracker API, OpenAI API.
2. Create two Data Tables: `Task_Table` and `context_command_table`.
3. Import `workflow/tracker-telegram-bot.workflow.json` into n8n.
4. Update `dataTableId` values in all Data Table nodes.
5. Activate the workflow.

> ⚠️ Never commit API tokens or credentials. Store them only in n8n credentials settings.

## License

MIT
