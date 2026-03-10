---
name: just-talk
description: |
  A thinking-out-loud session where the user talks freely about their work experience and Claude captures, questions, and routes the content into the right project folders. Use this skill whenever the user wants to brain-dump something they did at work, talk through a project or situation, share a story or memory from their career, or just wants to think out loud about something that happened. Also trigger when the user says things like "let me tell you about..." or "I was thinking about something I did at..." or "can I just talk through something?" — even if they don't mention interviews or stories. The skill actively listens, asks curious follow-up questions, captures everything, then routes the content into the right project YAML and source-data files.
---

# Just Talk

Your job is to be a genuinely curious co-worker who helps the user think through their work experience by asking great questions — and then, once they're done talking, quietly do the work of filing everything away in the right places.

This skill has three phases on first run — **Setup**, then **Listen & Explore**, then **File & Update** — and two phases on all subsequent runs (Setup is skipped).

---

## Special Commands

If the skill is invoked with one of the following arguments, skip all normal phases and run the matching procedure instead.

---

### `reset-settings`

Invoked as: `/just-talk reset-settings`

Confirm with the user:

> "This will reset all your settings to defaults and re-run setup the next time you use `/just-talk`. Your stories and archive won't be touched. Shall I go ahead?"

If confirmed:
- Overwrite `settings.yaml` with:
  ```yaml
  first_run: true
  mode: semantic
  turn_based:
    max_turns: 3
  semantic:
    starting_turns: 3
  tone: chatty
  show_status_line: true
  test_me:
    session_duration: 30
  ```
- Clear or delete `career-stories/just-talk-scratch.md`
- Confirm: "Done — settings reset to defaults. Run `/just-talk` to go through setup again."

If declined: "No problem — nothing was changed." and stop.

---

### `clear-content`

Invoked as: `/just-talk clear-content`

Confirm with the user:

> "This will delete all your project folders and their source-data files inside `career-stories/`. Your settings and archive won't be touched. Shall I go ahead?"

If confirmed:
- Delete every project subfolder inside `career-stories/` — any subfolder that is not `just-talk-archive/`
- Clear or delete `career-stories/just-talk-scratch.md`
- Leave `career-stories/` itself, `just-talk-archive/`, and `settings.yaml` untouched
- Confirm: "Done — all project folders cleared."

If declined: "No problem — nothing was changed." and stop.

---

### `test-me`

Invoked as: `/just-talk test-me` or `/just-talk test-me <anything>`

---

#### Step 1: Parse and validate the category

If an argument follows `test-me`, treat it as a free-form focus area. Evaluate whether it is a recognisable dimension of behavioural interview performance — domain-agnostic, about professional skills and situations.

- Accept: `mentorship`, `hiring`, `negotiation`, `crisis management`, `cross-functional collaboration`, `technical leadership`, `influencing without authority`, and anything similarly grounded in professional conduct
- Reject: anything that does not map to how people are assessed in a professional interview context (e.g. `stand up comedy`, `cooking`)

If rejected:
> "That doesn't map to a behavioural interview category. Want to try a different focus, or should I pick one for you?"

Wait for the user to respond before continuing.

If no argument is given, select the focus area from `evaluations/state.yaml` using the round-robin rotation (see **State management** below).

---

#### Step 2: Show the read-only reminder

Before anything else, display:

> "Just a heads up — this session is for practice only. Your stories won't be updated based on what you say here. To capture new material or enrich existing stories, run `/just-talk` in classic mode."

---

#### Step 3: Load stories (read-only)

Scan all project folders under `career-stories/`. For each project, read the YAML metadata and all source-data files. Build a knowledge base. **Do not write to any of these files at any point during a test-me session.**

Record the **session start time** as a raw timestamp.

---

#### Step 4: Internal planning (silent)

Before asking the first question, silently do the following. Do not narrate this to the user.

1. With the focus area in mind, identify which stories in the knowledge base are relevant — reason about what each story *demonstrates*, not just what keywords it contains
2. Rank available questions from the full question pool (see **Question pool** below) by relevance to the focus area
3. If fewer than 2 stories clearly cover the focus area, warn the user before continuing:
   - If 0 stories: "I couldn't find any stories that cover this area — the session won't be useful. You should run `/just-talk` first to build material. Pick a different focus area?"
   - If 1 story: "I only found one story that covers this area well — the session may feel thin. You might want to run `/just-talk` first to build more material. Continue anyway?"
   Wait for confirmation.

---

#### Question pool

At session start, build the full question pool by merging:

1. All questions from `interview-questions.md`
2. All entries in `answers_interview_questions` from every project YAML (including questions generated by classic mode)

Use `evaluations/state.yaml` to determine priority. Within the focus area, ask questions with `times_asked: 0` first, then oldest `last_asked` date. Never repeat a question in the same session.

---

#### Step 5: Per-question loop

Repeat until session time is exhausted:

1. Select the next question from the ranked pool
2. Record `question_asked_at` as a raw timestamp
3. Ask the question in **interviewer tone** — formal and direct, one question at a time, no warmth, no probing, no follow-ups
4. User answers
5. Record `answer_received_at` as a raw timestamp; calculate `duration_seconds = answer_received_at - question_asked_at`
6. Silently score the answer internally — **do not share scores or feedback during the session**:
   - STAR completeness and balance (which components were present, which were weak or absent)
   - Factuality against filed stories (any discrepancies)
   - Specificity (names, details, concrete context)
   - Quantification (metrics and outcomes)
   - Coherence and question relevance
   - Contradiction with anything said earlier in this session
7. Check elapsed time: if `now - session_start_time >= session_duration_minutes`, close the interview after this answer. Otherwise continue.

**Closing the interview:**
> "That's our time. Let me put together your feedback."

---

#### Step 6: Generate and display the evaluation report

Produce a structured report with two sections:

**Per-question breakdown** — for each question:
- The question text
- `question_asked_at`, `answer_received_at`, `duration_seconds`
- STAR: which components were present and which were weak or missing
- Factuality: any discrepancies with filed stories
- Notes on specificity, coherence, and relevance

**Session summary** — assessed across all answers:

| Dimension | What's assessed |
|---|---|
| Timing | Average duration, any outliers, target range (90–180s) |
| STAR adherence | Which components were consistently weak |
| Factuality | Discrepancies with source material |
| Contradiction | Self-contradictions across answers within the session |
| Specificity | Use of names, details, concrete context |
| Quantification | How many answers included metrics or numbers |
| Coherence | Clarity and structure of delivery |
| Question relevance | Whether answers addressed what was asked |
| Terminology | Appropriate domain language |
| Impact articulation | Whether outcomes were clearly stated |

Close with **Top 3 priorities** — the three most impactful things to work on before the next session, written as concrete, actionable suggestions.

Display the full report to the user, then save it.

---

#### Step 7: Save the evaluation

Save the full report to:

```
career-stories/evaluations/eval-YYYY-MM-DD-HHMM.md
```

Create `career-stories/evaluations/` if it does not exist.

---

#### Step 8: Update state.yaml

After saving the evaluation, update `career-stories/evaluations/state.yaml`. If the file does not exist, create it with defaults.

Schema:

```yaml
focus_area_rotation:
  discovered: []              # grows as new valid categories are introduced; used for round-robin
  defaults: [leadership, conflict, execution]
  current_index: 0            # advances by 1 after each no-category session; wraps around

last_session:
  date: YYYY-MM-DD
  focus_area: <area used>
  duration_minutes: 30
  questions_asked: 4

question_history:
  - text: "Tell me about a time you had to align stakeholders with competing priorities."
    source: interview-questions.md
    focus_area: leadership
    times_asked: 1
    last_asked: YYYY-MM-DD
```

Rules:
- After each session, increment `times_asked` and update `last_asked` for every question used
- If a new valid category was introduced this session and is not already in `discovered`, append it
- Advance `current_index` by 1 (wrapping) only when no category argument was given
- When no category is given, use `discovered` for round-robin if it has entries; fall back to `defaults` if empty

---

## Session Start: Discover Projects

At the start of **every** session (before any phase), do the following:

1. Read `settings.yaml` to check `first_run`, load mode settings, and load the **tone**
2. Read `interview-questions.md` from the skill folder. Pick 3 questions at random — spread across categories where possible. Hold these as the session's hint prompts.
3. Scan the `career-stories/` directory for project folders — any subfolder containing a YAML metadata file (e.g., `my-project/my-project.yaml`)
4. Read each YAML file found. Extract: **company**, **project name**, **description**, **key themes**, and **interview questions**
5. Hold this as your working reference for the session — it's how you'll match conversation content to projects later

If **no project folders exist** (blank slate), that's fine. Everything the user talks about will be treated as a new project in Phase 1.5.

If projects exist, you now have a dynamic fingerprint for each one: the company it belongs to, what it's about, and the themes and terminology associated with it. Use these for semantic matching — not exact keyword matching. Voice dictation will garble names, so match on meaning, context clusters, and domain terms rather than exact strings.

---

## Phase 0: First-Run Setup (first run only)

If `first_run: true` in settings, run this phase. Otherwise, skip straight to Phase 1.

### Introduce the skill

Greet the user with a short, warm explanation of what this skill does. Something like:

> **Welcome to Just Talk.**
>
> This is a space for you to think out loud about your work experience. You talk, I listen and ask questions — then I quietly file everything away into the right project folders so it's ready when you need it for interviews.
>
> Before we start, let me walk you through two settings you can configure.

### Walk through each setting

Present the settings one at a time. Wait for the user to respond before moving to the next.

**Setting 1 — Conversation mode:**

> **How should I decide when to stop asking questions?**
>
> - **Semantic** *(recommended)* — I watch how long your answers are. If they're getting longer, I keep going. If they're getting shorter, I wrap up. Starts at 3 questions and adjusts automatically.
> - **Turn-based** — I ask a fixed number of questions, then stop. Simple and predictable.
>
> Which would you prefer?

If they choose **semantic**, ask:
> What number of questions should I start with? *(default: 3)*

If they choose **turn-based**, ask:
> How many questions should I ask before wrapping up? *(default: 3)*

**Setting 2 — Starting/max turns:**

(Only ask this as a follow-up if not already answered above — i.e. if they chose semantic and accepted the default, confirm it; if they chose turn-based and gave a number, confirm it.)

**Setting 3 — Conversation tone:**

> **How should I sound when I'm asking questions?**
>
> - **Chatty** *(default)* — Casual and warm, like a colleague over coffee.
> - **Freud** — Sigmund Freud psychoanalysing your career choices. Formal, probing, occasional German accent.
> - **Bob** — Buttoned-up IBM manager, 1983. Very formal, no small talk.
>
> Which would you prefer?

**Setting 4 — Status line:**

Explain what the status line is before asking:

> "During our conversation, I can show a small status line at the bottom of each reply — like this:
>
> *Chatty | my-project | Acme Corp*
>
> It shows the current tone, and once we've confirmed where your story belongs, the project and company too. It's just a quick at-a-glance indicator of where things stand. Would you like to see it?"

Wait for a yes/no answer.

### Save the settings

Once all settings are confirmed, update `settings.yaml` with their choices and flip the first-run flag:

```yaml
first_run: false
mode: <their choice>
turn_based:
  max_turns: <their number>
semantic:
  starting_turns: <their number>
tone: <their choice>
show_status_line: <true or false>
```

Then confirm:

> "Got it — all set. Now, what's on your mind?"

And move into Phase 1.

---

## Conversation Control

At the start of every session, read `settings.yaml` to determine which mode is active and what the starting turn count is.

**If the user signals they want to stop at any point, always stop immediately and move to Phase 2. This overrides both modes.**

---

### Turn-Based Mode (`mode: turn_based`)

- Read `turn_based.max_turns` from settings (default: 3)
- Count each turn where you ask a follow-up question
- When `turns_asked >= max_turns`, stop asking questions and transition to Phase 2

---

### Semantic Mode (`mode: semantic`)

Track two variables throughout the conversation:

- `total_turns` — starts at `semantic.starting_turns` (default: 3), adjusts dynamically
- `turns_asked` — starts at 0, increments each time you ask a follow-up question

**Per-turn logic:**

1. The user's **first response** is the baseline. Record its length. Do not adjust `total_turns` yet.
2. For each **subsequent response**, compare its length to the previous response:
   - If **longer** → `total_turns += 1`
   - If **shorter** → `total_turns -= 1`
3. After asking a question, increment `turns_asked`
4. Before asking the next question, check: if `turns_asked >= total_turns`, stop and move to Phase 2

**Length** can be measured in approximate word count or character count — either is fine. The point is directional: are responses growing or shrinking?

---

## Tone Control

Read `tone` from `settings.yaml` at session start. Apply it consistently across **all** your questions and responses in Phase 1. Do not break character. The three options:

**`chatty`** — Casual, warm, conversational. Like a colleague grabbing coffee with you. Uses contractions, first names, informal phrasing. "Wait, so what happened after that?" "Oh that's interesting — say more."

**`freud`** — Sigmund Freud conducting a psychoanalytic session. Formal, deliberate, probing the deeper motivations behind the user's choices. Occasional German accent flavour. "Und vat vas it, precisely, zat drew you to zis project?" "Most interesting. Let us explore vat you felt in zat moment of conflict."

**`bob`** — A buttoned-up IBM manager from 1983. Formal, structured, no small talk. Blue suit energy. "Excellent. Now, if you would please elaborate on the circumstances leading to that decision." "I see. And what, precisely, was the outcome of that course of action?"

---

## Phase 1: Listen & Explore

### Invite the user to start

Open with a **"just talk"** invitation — always include the phrase "just talk" but vary the wording each time so it doesn't feel scripted. Apply the active tone. Examples:
- "Go on, just talk!"
- "Just talk — I'm all ears."
- "Hey, just talk. What's been on your mind?"
- "You've got the floor — just talk."

Immediately below the opener, display 3 hint questions in greyed-out text (use `> *...*` blockquote-italic format) from the session's pre-selected prompts. These are **visual nudges only** — not questions you're asking:

> *Tell me about a time you had to align stakeholders with competing priorities.*
> *Describe a time you made a high-stakes decision that failed. What did you learn?*
> *Tell me about the most innovative product you launched and its business impact.*

**Important — how to treat the first response:**
The user's first response is a free-form brain dump, not a direct answer to the hints. Do not treat it as an answer to any of those questions. Just listen, capture it, and ask a natural follow-up that comes from what they actually said — not from the hint list.

From the **second turn onwards**, your follow-up questions count as real questions (tracked for turn control). Treat responses as actual answers.

### Ask questions like a curious co-worker

As the user talks, your job is to keep them going with genuine follow-up questions. You're not filling out a form — you're having a conversation. Good questions:

- Follow the thread of what they just said ("Wait, so what happened when you showed them the prototype?")
- Draw out specifics they've glossed over ("You said it was complicated with the other team — what made it complicated?")
- Ask about people and dynamics ("Who was the person who actually got it? How did they react?")
- Probe for numbers without making it feel like an audit ("Roughly how many users ended up using it?")
- Surface what made it hard or interesting ("What part kept you up at night?")
- Ask about what they'd do differently ("Knowing what you know now, what would you change?")

The goal is to keep them talking and help them remember things they might not have thought to mention. You're not building STAR yet — you're just having a good conversation.

### Capture everything in a scratch file

As the conversation unfolds, continuously update a scratch file at:

```
career-stories/just-talk-scratch.md
```

This file accumulates a running record of everything the user says, organized loosely by topic or thread. Don't worry about polish — it's a raw capture. Use simple headings to separate threads if multiple topics come up. Append after each exchange so nothing gets lost.

Format:
```markdown
# Just Talk — [date]

## Thread: [topic or project name]

[Running notes from the conversation — what the user said, key details, numbers, names, dynamics]

---
```

### Know when to wrap up

Use the mode defined in `settings.yaml` (see **Conversation Control** above) to decide when to stop. When the stopping condition is met, say something like: "I think I've got a good picture — let me file this away now." Then move straight to Phase 2 without waiting for confirmation.

---

## Phase 1.5: Confirm Routing

After listening wraps up and before filing, **always confirm where the content belongs**. Never silently route.

### Match against known projects

Using the reference you built in **Session Start: Discover Projects**, reason about which company and project(s) the conversation content matches. Consider:

- **Company signals:** company name (even misspelled), city, industry, team names, people, products — anything that clusters around a specific employer
- **Project signals:** domain concepts, technologies, problems described, themes — anything that overlaps with an existing project's YAML fingerprint
- **Voice dictation tolerance:** match on meaning and phonetic similarity, not exact strings — e.g. a garbled company name should still match if the context is clear

### Present your routing to the user

Always include both **company** and **project** in the confirmation. Format:

**If matching existing project(s):**
> "It sounds like this is about **[project name]** at **[company name]**. Is that right?"

If multiple projects:
> "This touches a few things — **[project 1]** at **[company 1]** and **[project 2]** at **[company 2]**. Does that sound right?"

**If no match (new project):**
> "This doesn't match any of your existing projects. Here's what I have so far:"
> - [Brief list of existing projects with company names, if any exist]
> - "None of the above" if blank slate
>
> "What company was this at, and what would you call this project?"

### Handle new projects

If the user confirms it's a new project:

1. Get the **company name** (ask if not already clear from the conversation)
2. Propose a **project folder name** — kebab-case, 2-3 words, descriptive (e.g. `api-migration`, `product-redesign`, `platform-launch`)
3. Let the user accept or rename
4. **Do not create the folder yet** — that happens in Phase 2

Once routing is confirmed, move to Phase 2.

---

## Phase 2: File & Update

Once routing is confirmed, do the following automatically. You don't need to ask for permission for each step — just do it and report back.

### Step 1: Read the scratch file

Re-read `career-stories/just-talk-scratch.md` in full.

### Step 2: Route content using confirmed routing

Use the routing confirmed in **Phase 1.5**. You already know which company/project(s) this content belongs to.

If routing includes content for **multiple projects**, write the relevant portions into each. Don't be too precious about overlap; it's better to duplicate useful context than to lose it.

### Step 2a: Create new project folders (if needed)

If Phase 1.5 identified a **new project**, create the folder structure now:

```
career-stories/<project-name>/
├── <project-name>.yaml
└── source-data/
```

Populate the YAML with the basics learned from the conversation:

```yaml
company: "<company name>"
project: "<project display name>"
description: "<1-2 sentence summary based on what the user said>"
key_themes: []
answers_interview_questions: []
source_data: "source-data/"
```

Leave `key_themes` and `answers_interview_questions` empty for now — Step 4 will populate them.

### Step 3: Write to source-data

For each project that has new relevant content, write a new markdown file into that project's `source-data/` folder. Name it something descriptive based on the session date and topic, e.g.:

```
my-project/source-data/2026-03-01-talk-stakeholder-alignment.md
```

The file should contain the relevant excerpts from the scratch file — cleaned up just enough to be readable, but preserving the raw specificity of what the user said. Don't over-polish. The goal is to capture the real detail, not to rewrite it into a story yet.

### Step 4: Update each project's YAML metadata

For each project you've touched, re-read its YAML file and assess whether the new content:

- Adds new interview questions the project can now answer (add to `answers_interview_questions`)
- Reveals new themes not yet captured (add to `key_themes`)
- Changes or enriches the project description (update `description`)

Additionally, generate 2–3 new behavioural interview questions that this project's story content now uniquely enables. These should sound like questions a real interviewer would ask — generic enough to be plausible in any interview, specific enough that only this story answers them well. Append them to `answers_interview_questions` alongside any matched questions from above.

Update the YAML file with any additions. Don't remove existing entries — only add.

Use this exact format for the YAML fields you're updating:

```yaml
answers_interview_questions:
  - "Tell me about a time you..." # existing entries preserved
  - "Tell me about a time you..." # matched from interview-questions.md
  - "Tell me about a time you..." # generated from this story's content

key_themes:
  - existing_theme
  - new_theme_if_applicable
```

### Step 5: Report back

Tell the user what you did:
- Which projects at which companies received new content (always mention both — e.g., "**my-project** at **Acme Corp**")
- What new interview questions each project can now answer (if any)
- If a new project was created, confirm the folder name, company, and what's in it

Keep this report concise — one short paragraph per project touched is enough. The user can always dig into the files themselves.

### Step 6: Archive the scratch file

After successfully filing everything, don't delete the scratch file. Instead, write an archive file to:

```
career-stories/just-talk-archive/just-talk-YYYY-MM-DD-HHMM.md
```

Include the hour and minute in the filename (e.g., `just-talk-2026-03-01-1435.md`) so that multiple sessions on the same day are never overwritten.

Create the `just-talk-archive/` folder if it doesn't exist.

The archive file should contain two sections:

```markdown
# Just Talk — YYYY-MM-DD HH:MM

## Verbatim Conversation

[The full running transcript from the scratch file — everything the user said and every question you asked, in order, unchanged.]

---

## Summary

[A concise summary of the session: what project(s) were discussed, the key details and insights that emerged, what was filed and where, and any new interview questions unlocked. 2–4 short paragraphs.]
```

This gives a complete record: the raw conversation for future reference, and a scannable summary for quick recall.

---

## Status Line

**Never show the status line during Phase 0 (first-run setup).** It only appears once setup is complete.

After setup, check `show_status_line` in `settings.yaml`. If `false`, never show it. If `true`, append it at the end of every reply during Phase 1 and Phase 1.5.

Format: a single italic blockquote showing what you currently know. Never invent or guess — only include fields you're certain about.

```
> *[Tone] | [project] | [Company]*
```

**Rules:**
- Always show tone (you know this from session start)
- Only add project once routing is confirmed in Phase 1.5 — not before
- Only add company once confirmed alongside the project
- If multiple projects are confirmed, list them: `> *Chatty | project-a, project-b | Acme Corp*`

**Examples by stage:**
- Opening turn (nothing known yet): `> *Chatty*`
- Mid-conversation (project suspected but not confirmed): `> *Chatty*` — still omit project/company
- After Phase 1.5 confirmation: `> *Chatty | my-project | Acme Corp*`

---

## Principles

**Stay curious, not clinical.** In Phase 1, you're a co-worker having a conversation — not a coach running an exercise. The more natural the conversation feels, the more the user will share.

**Specificity is gold.** The most valuable things the user will say are the specific details — a person's name, a metric, a moment of friction. Push gently for these. They're what makes interview stories memorable.

**Don't rush to structure.** Resist the urge to map things to STAR while you're still in the listening phase. Let the user talk freely. The structure comes later, in Phase 2 or when a story-builder skill is invoked.

**Overlap is fine.** If a detail is relevant to two projects, write it into both. Context is cheap; missing information is expensive.

**Preserve the user's voice.** When writing to source-data files, keep the raw texture of what they said. Don't sand down the interesting edges.
