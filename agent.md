# Agent Reference — next-vocal

> **For AI agents only.** Read this file once, then read `progress.lock`. That is all you need to act correctly. You do not need to read any other file in this repo.

---

## What This Repo Is

A personal English-speaking practice journal. Every day a new session is generated containing:

- A **speaking task** (`task.md`) with exercises tailored to build native-like fluency.
- A blank **notes file** (`notes.md`) for the owner to fill in after practising.

A GitHub Action runs at **midnight UTC** to create these files automatically. When it has not run (or needs to be done manually), the agent must do it instead.

---

## The Only File You Need to Read: `progress.lock`

`progress.lock` is the single source of truth. Here is what each field means:

```jsonc
{
  "last_updated": "YYYY-MM-DD",          // date progress.lock was last written (not when the owner practised)

  "current_session": {
    "date":        "YYYY/MM/DD",         // folder path of TODAY's session
    "folder":      "YYYY/MM/DD",         // same as date
    "topic":       "...",                // today's English skill topic
    "skill_focus": ["...", "..."],       // bullet skills for today
    "completed":   false                 // true once the session files exist
  },

  "next_session": {
    "scheduled_date": "YYYY/MM/DD",      // tomorrow's folder path
    "folder":         "YYYY/MM/DD",
    "prompt":         "..."              // full AI prompt to generate tomorrow's task
  },

  "settings": { ... },                   // fixed — do not change
  "history":  [ ... ]                    // append-only log of past sessions
}
```

**How to determine what needs to be done:**

1. Check `current_session.date` vs today's date (UTC).
2. If they **match** and the folder `current_session.folder/task.md` **does not exist** → generate today's session (see below).
3. If they **do not match** (lock is behind) → the Action missed a day; treat `next_session` as today, generate it, and update the lock.
4. If `task.md` **already exists** for today → nothing to do.

---

## Folder & File Structure

```
YYYY/
  MM/
    DD/
      task.md    ← AI-generated daily task (you create this)
      notes.md   ← blank practice-notes template (you create this)
progress.lock    ← update after creating the files
agent.md         ← this file (never modify)
README.md        ← human-facing docs (never modify)
.github/
  workflows/
    daily-task.yml  ← the GitHub Action (never modify)
```

---

## Step-by-Step: How to Generate a Daily Session

### 1 — Read `progress.lock`

Identify:
- `TODAY_FOLDER` = `current_session.folder` (e.g. `2026/02/26`)
- `TODAY_TOPIC`  = `current_session.topic`
- `SKILL_FOCUS`  = `current_session.skill_focus` (array of strings)

### 2 — Create `TODAY_FOLDER/task.md`

Use this exact structure (copy the style of any existing `task.md`):

```markdown
# 📅 YYYY/MM/DD — <Topic>

> **Skill Focus:** Skill 1 · Skill 2 · Skill 3 · Skill 4

---

## 🎯 Today's Goal

One or two sentences describing what the learner will achieve.

---

## 📖 Explanation

Explain the skill in plain, engaging language. Use bullet points for key tools/patterns.
Bold the tool names (e.g. **Hedges**, **Starters**).

---

## 🗣️ 10 Example Sentences (Textbook vs Native)

| # | ❌ Textbook | ✅ Native |
|---|---|---|
| 1 | "..." | "..." |
...
| 10 | "..." | "..." |

---

## ⏱️ 5-Minute Speaking Drill

Set a 5-minute timer. Speak aloud (record yourself if possible):

1. **[1 min]** Exercise 1
2. **[1 min]** Exercise 2
3. **[1 min]** Exercise 3
4. **[1 min]** Exercise 4
5. **[1 min]** Exercise 5

---

## ⚠️ 3 Common Mistakes to Avoid

1. **Mistake title** — explanation.
2. **Mistake title** — explanation.
3. **Mistake title** — explanation.

---

## 🎲 Challenge Sentence

Try saying this out loud — naturally, without pausing:

> *"..."*

Aim to deliver it smoothly with natural rising-and-falling intonation, as if you're mid-conversation.

---

## 📝 Your Notes

> *(Fill in `notes.md` after completing today's practice.)*
```

### 3 — Create `TODAY_FOLDER/notes.md`

Always use this exact template:

```markdown
# 📝 Practice Notes — YYYY/MM/DD

> Fill in this file after completing today's task.

## Speaking Drill Reflection

<!-- What went well? What felt unnatural? -->

## Sentences I Want to Remember

<!-- Copy any example sentences or phrases you want to keep -->

## My Own Examples

<!-- Write 3–5 of your own sentences using today's patterns -->

## Questions / Confusions

<!-- Anything you want to look up or ask about -->
```

### 4 — Generate the next session's prompt

At the end of `task.md`, the AI is expected to provide a JSON block with the next session's metadata:

```json
{
  "next_topic": "...",
  "next_skill_focus": ["...", "..."],
  "next_prompt": "..."
}
```

When generating manually, craft `next_prompt` following this pattern:

> *"You are an expert English fluency coach helping a non-native speaker sound like a native. The learner has strong grammar. Yesterday's topic was '**\<TODAY_TOPIC\>**'. Today (**\<TOMORROW_DATE\>**) focus on: **\<NEXT_TOPIC\>** — \<one-sentence description\>. Provide: (1) a short explanation of the skill, (2) 10 example sentences contrasting textbook vs native phrasing, (3) a 5-minute speaking drill, (4) 3 common mistakes to avoid, (5) a fun challenge sentence to try out loud. End by suggesting the NEXT topic that logically follows, formatted as a JSON block: {\"next_topic\": \"...\", \"next_skill_focus\": [\"...\"], \"next_prompt\": \"...\"}."*

### 5 — Update `progress.lock`

Make these changes atomically:

| Field | New value |
|---|---|
| `last_updated` | today's date as `YYYY-MM-DD` |
| `current_session` | move `next_session` values here; set `completed: false` (means files exist but practice not yet done — it becomes `true` in `history` when the next day's Action runs) |
| `next_session.scheduled_date` | tomorrow's date as `YYYY/MM/DD` |
| `next_session.folder` | same as `scheduled_date` |
| `next_session.prompt` | the new prompt you crafted in step 4 |
| `history` | append the **previous** `current_session` with `completed: true` (avoid duplicates — check `date`) |

---

## Commit Message Convention

```
chore: daily task for YYYY/MM/DD
```

---

## What NOT to Do

- Do **not** modify `agent.md`, `README.md`, or `.github/workflows/daily-task.yml`.
- Do **not** edit a `notes.md` that already has content written by the owner.
- Do **not** create files for future dates — only today's session.
- Do **not** change `settings` inside `progress.lock`.
- Do **not** remove entries from `history`.
