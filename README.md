# next-vocal

A personal English-speaking practice journal powered by AI.

## Goal

I have strong grammar knowledge and want to **speak like a native English speaker** — natural rhythm, colloquial expressions, idiomatic phrasing, and confident fluency.

## How It Works

1. **Daily Tasks** — Every day at midnight a GitHub Action generates a new speaking task using GitHub Copilot (AI). The task is tailored to build on the previous session and push the next skill level.
2. **Dated Folders** — Each session lives in a folder named `yyyy/mm/DD` (e.g. `2026/02/25/`). Inside you'll find:
   - `task.md` — the AI-generated speaking prompt and exercises for that day.
   - `notes.md` — your own practice notes, recordings, or reflections (fill this in yourself).
3. **Lock File** — `progress.lock` is the single source of truth for the project. It tracks:
   - The current session date and topic.
   - The exact AI prompt that will be used to generate **tomorrow's** task.
   - A history of all past sessions.
   - The folder structure convention.

## Folder Structure

```
yyyy/
  mm/
    DD/
      task.md    ← AI-generated daily task
      notes.md   ← your practice notes
progress.lock    ← machine-readable project state
```

## Workflow

```
Midnight UTC
   │
   ▼
GitHub Action reads progress.lock
   │
   ▼
Calls GitHub Models API (gpt-4o) with the locked "next_prompt"
   │
   ▼
Creates yyyy/mm/DD/task.md with today's exercises
   │
   ▼
Updates progress.lock with new session info & next prompt
   │
   ▼
Commits & pushes → loop begins again tomorrow
```

## Getting Started

Fork this repo, enable GitHub Actions, and the daily cycle starts automatically.  
Fill in `notes.md` each day after completing the task.
