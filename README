Tracker Telegram Bot for n8n
An n8n automation workflow that connects a Telegram bot to Yandex Tracker.
Users create and update tasks via natural language messages in Telegram. AI parsing extracts task content and deadlines automatically.

Overview
The workflow connects Telegram, n8n, OpenAI, and Yandex Tracker into one task management flow.
The bot accepts commands, maintains per-user dialog context between messages, creates issues in Yandex Tracker, and stores synchronization data in local Data Tables.

Workflow Schema
Full n8n workflow: from incoming Telegram message to task creation and update in Yandex Tracker

Use Case
Instead of opening Yandex Tracker manually, the user sends a message in Telegram such as /newtask call the client, deadline tomorrow at 10:00. The workflow parses the text, creates the issue in Yandex Tracker, and saves the task metadata locally in Task_Table.

Architecture
Component	Tool
Automation engine	n8n
Messenger	Telegram Bot API
Task tracker	Yandex Tracker API
AI parsing	OpenAI (GPT)
Data storage	n8n Data Tables
The workflow processes each message through the following pipeline:

Telegram Trigger — receives the incoming message.

Command parser — detects /start, /newtask, /updatetask, /list, /cancel.

Context storage — reads and writes current user dialog state from context_command_table.

AI parser — extracts structured task content and deadlines from free-form text.

Yandex Tracker integration — creates or updates issues via Yandex Tracker API.

Task storage — saves and updates task metadata in Task_Table.

Data Model
Task_Table
Stores all tasks created through the bot, synchronized with Yandex Tracker.

Task Table
Example data in Task_Table: task content, deadline, Tracker_Key, sync status

Field	Type	Description
chatid	number	Telegram chat ID of the user
TaskContent	string	Task description
DeadlineAt	dateTime	Task deadline in ISO 8601
Finished	boolean	Whether the task is marked as finished
notified	boolean	Whether the user has been notified
Tracker_Task_ID	string	Yandex Tracker internal object ID
Tracker_Key	string	Human-readable issue key, e.g. RAZVITIE-15
LastSyncStatus	string	Last sync result: created, finished, etc.
context_command_table
Stores the current dialog state for each Telegram user. Allows multi-step interactions across separate messages.

Context Command Table
Example data in context_command_table: active command and step per user

Field	Type	Description
chatid	number	Telegram chat ID of the user
command	string	Active command, e.g. updatetask
step	string	Current step in the dialog, e.g. wait_updatetask_payload
draft_taskcontent	string	Temporary task content draft
draft_deadlineat	string	Temporary deadline draft
draft_taskid	string	Task ID being edited
draft_action	string	Pending action: finish, changdeadline, remove
updated_at	dateTime	Last update to this context record
Task Identifiers
Each task has three identifiers — it is important not to confuse them:

ID	Field	Example	Usage
Local DB row ID	id	1	Internal n8n table reference
Yandex Tracker object ID	Tracker_Task_ID	6a33d392ed24231007ea34ea	Used in API calls to Yandex Tracker
Human-readable issue key	Tracker_Key	RAZVITIE-15	Shown to users in Telegram messages
Yandex Tracker Queue
Yandex Tracker
Tasks created by the bot appear in the configured Yandex Tracker queue

Commands
Command	Description
/start	Shows welcome message
/newtask	Starts task creation flow
/updatetask	Starts task update flow
/list	Shows active tasks
/cancel	Cancels current operation and clears dialog context
Workflow Behavior
Create Task (/newtask)
User sends /newtask with task description.

AI parses task text and deadline.

Issue is created in Yandex Tracker.

Task metadata is saved to Task_Table.

Bot replies with the Tracker key and deadline confirmation.

Update Task (/updatetask)
User sends /updatetask.

Bot asks for the task to update (by number from /list).

User selects an action: finish, change deadline, or remove.

Changes are applied in Yandex Tracker.

Task_Table is updated with the new sync status.

Cancel (/cancel)
The bot deletes the active row from context_command_table and confirms cancellation.
Yandex Tracker and Task_Table are not affected.

Deadline Format
Deadlines must be in ISO 8601 format with Moscow time (UTC+3):

text
2026-06-20T10:00:00.000+03:00
The AI node converts user input (e.g. "tomorrow at 10:00") into this format automatically.
Yandex Tracker rejects any other date format.

Requirements
n8n instance (self-hosted or cloud)

Telegram bot token

Yandex Tracker OAuth token and organization ID

OpenAI API key

Two configured n8n Data Tables: Task_Table and context_command_table

Setup
Create credentials in n8n:

Telegram API (bot token)

Yandex Tracker API (OAuth token + org ID)

OpenAI API (API key)

Create two Data Tables in n8n:

Task_Table — with fields listed in the Data Model section above

context_command_table — with fields listed in the Data Model section above

Import the workflow:

Go to n8n → Workflows → Import

Select workflow/tracker-telegram-bot.workflow.json

Update workflow configuration:

Replace dataTableId references in all Data Table nodes with your actual table IDs

Assign the correct credentials to each node

Activate the workflow.

Repository Structure
text
tracker-telegram-bot-n8n/
├── README.md
├── README.ru.md
├── LICENSE
├── .gitignore
├── workflow/
│   └── tracker-telegram-bot.workflow.json
└── docs/
    ├── workflow-schema.jpg
    ├── task-table.jpg
    ├── context-command-table.jpg
    └── yandex-tracker.jpg
Notes
Do not commit API tokens or credentials to the repository.

Add .gitignore with *.env and any local config files.

The Tracker_Task_ID is used for API calls; Tracker_Key is used for display only.

License
MIT
