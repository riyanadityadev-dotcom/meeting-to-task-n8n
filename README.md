# Meeting-to-Task Automation with n8n

An n8n workflow that turns a meeting transcript into a structured task list. Before anything is saved, a human must approve the list by email. Approved tasks are saved to Google Sheets, and every run is written to a log.

> Demo project with fictional data. Not client work.

## Demo

[Watch the demo video](https://www.loom.com/share/aec8c89f7b814478a7703e3db74a257d) (narrated in Indonesian)

[Demo Meeting-to-Task]<img width="1596" height="910" alt="Thumbnail Demo Workflow" src="https://github.com/user-attachments/assets/f0d6c176-5281-4f41-8269-c3656028e26b" />

## Problem

Tasks from meetings are often recorded late or missed entirely. AI-generated summaries can also be wrong. This workflow extracts tasks automatically, but it does not save anything until a human approves it.

## Who is involved

- **Form submitter:** the person who holds the meeting transcript, for example the note-taker. They paste the transcript into the form.
- **Approver:** the person responsible for the meeting outcome, for example a project manager. They receive an email with the summary and task list, then approve or reject it.
- **Task owner:** the person named in the transcript. The AI records the name in the `owner` column.

The form submitter and the approver can be the same person.

## How it works

Form → LLM → Validation → Human approval → Google Sheets → Log

1. **Form:** the submitter enters the meeting title, date, and transcript.
2. **LLM (Gemini):** the model produces a summary and a task list in JSON format. Each task contains the task, owner, due date, priority, source quote, and a `needs_review` flag.
3. **Validation:** a Code node checks the JSON structure. Empty or malformed output is rejected.
4. **Human approval:** the approver receives an email and chooses approve or reject.
5. **Storage:** approved tasks are appended to the `Tugas` sheet.
6. **Log:** the result of every run is written to the `Log` sheet.

The prompt forbids the model from guessing. If the owner or due date is not mentioned in the transcript, the value is set to "tidak disebutkan" (not mentioned). Tasks the model is unsure about are marked "PERLU DICEK" (needs review) in the approval email.

> Note: the sheet names (`Tugas` = tasks) and the status values in the log are kept in Indonesian, matching the workflow as built.

## Scope

This project covers the middle part of the following meeting automation chain:

Meeting → Transcript → Summary → Action Items → Assignment → Follow-up

Status of each stage:

- **Meeting → Transcript:** not covered. The transcript is pasted manually through the form.
- **Summary:** done.
- **Action Items:** done, as structured and validated JSON.
- **Assignment:** partial. Task owners are recorded in Sheets, but they are not notified automatically after approval.
- **Follow-up:** not covered.

Two key design decisions:

- **Human approval:** the AI only proposes tasks. Nothing is saved until the approver accepts them.
- **Run log:** every run is recorded, whether it succeeded, failed, or was rejected. The log can be used to monitor workflow performance.

## Error handling

What happens when something goes wrong:

- **LLM call fails or times out:** retried up to 4 times (5-second interval), then logged as "gagal LLM" (LLM failed).
- **LLM output is empty or invalid:** logged as "gagal validasi" (validation failed) with the error message.
- **Approver rejects or does not respond:** logged as "ditolak atau tidak ada respon" (rejected or no response).
- **Saving to Sheets fails:** logged as "gagal simpan" (save failed).

## Setup

1. Import the file `Demo_AI_Meeting-to-Task.json` into n8n.
2. Create three credentials and attach them to the matching nodes:
   - **Header Auth** for the LLM node. Set the header name to `x-goog-api-key` and the value to your own Gemini API key.
   - **Gmail OAuth2** for the approval node.
   - **Google Sheets OAuth2** for the storage and log nodes.
3. Create one spreadsheet with two sheets:
   - `Tugas`, row 1: `task, owner, due_date, priority, source_quote, needs_review, meeting_title, meeting_date`
   - `Log`, row 1: `waktu, judul_meeting, status, jumlah_action_items, detail`
4. Replace `GANTI_DENGAN_ID_SPREADSHEET` with your spreadsheet ID. Replace `GANTI_DENGAN_EMAIL_PENYETUJU` with the approver's email address.
5. Open the form, paste a **fictional** transcript, and run the workflow.

The values you need to prepare are listed in the `.env.example` file.

## Tech stack

n8n, Gemini API, Gmail, Google Sheets

## Limitations and roadmap

- Only one approver, and approval works through email only.
- Tested only with fictional transcripts.
- Planned improvements: audio-to-transcript conversion, sending tasks to their owners after approval (for example via email or Slack), and scheduled follow-up reminders.
