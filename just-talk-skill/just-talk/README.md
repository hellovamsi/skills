# Just Talk — A Claude Code Skill for Career Story Capture

**Just Talk** is a Claude Code skill that helps you think out loud about your work experience and automatically files everything into structured project folders — ready to use for behavioral interview prep.

You talk. Claude listens, asks good follow-up questions, then quietly organizes everything into the right place.

---

## What It Does

- Opens a relaxed conversation with "just talk" prompts and optional hint questions
- Asks curious follow-up questions in your chosen tone (casual, Freud, or IBM Bob)
- Knows when to stop — either after a fixed number of turns, or by sensing when your answers are getting shorter
- Confirms where the content belongs (company + project) before filing anything
- Creates project folders on the fly if the story is new
- Writes raw session notes to `source-data/`, updates project YAML metadata, and archives the full conversation

---

## Requirements

- [Claude Code](https://claude.ai/code) installed and running
- A `career-stories/` working directory (can start completely empty)

---

## Installation

1. Copy the `just-talk/` folder into your project's `.claude/skills/` directory:

```
your-project/
└── .claude/
    └── skills/
        └── just-talk/
            ├── SKILL.md
            ├── settings.yaml
            └── interview-questions.md
```

2. That's it. Run it with:

```
/just-talk
```

The first time you run it, it will walk you through a short setup to configure your preferences — mode, tone, and whether to show the status line.

---

## Configuration

All settings live in `settings.yaml`. You can edit them directly at any time, or reset `first_run: true` to re-run the setup wizard.

```yaml
first_run: true         # set to true to re-run the setup wizard

mode: semantic          # semantic | turn_based
turn_based:
  max_turns: 3          # stop after this many questions
semantic:
  starting_turns: 3     # starting number of questions; adjusts based on response length

tone: chatty            # chatty | freud | bob

show_status_line: true  # show/hide the status line at the bottom of each reply
```

**Stopping modes:**
- `semantic` — watches how long your answers are; keeps going if they're growing, wraps up if they're shrinking
- `turn_based` — asks a fixed number of questions, then stops

**Tones:**
- `chatty` — casual, warm, like a colleague over coffee
- `freud` — Sigmund Freud psychoanalysing your career choices
- `bob` — buttoned-up IBM manager, 1983, very formal

**Status line:**
When enabled, a small indicator appears at the bottom of each reply during the conversation:

> *Chatty | my-project | Acme Corp*

It shows the active tone, and once confirmed, the project and company. Set `show_status_line: false` to hide it.

---

## test-me — Practice Mode

Run a timed mock interview against your own filed stories:

```
/just-talk test-me
/just-talk test-me mentorship
/just-talk test-me cross-functional collaboration
```

With no argument, Claude picks a focus area using round-robin rotation. With an argument, Claude validates it as a behavioural interview category and focuses the session on it.

**What it does:**
- Asks questions like an interviewer — formal, direct, no follow-ups
- Times each answer (raw timestamps + duration recorded)
- Silently evaluates every answer against STAR format, factuality, coherence, contradiction, and more — nothing is shown during the interview
- Ends the session when the configured time limit is reached
- Produces a per-question breakdown and an overall session summary with a Top 3 priorities list
- Saves the evaluation to `career-stories/evaluations/`

**Important:** test-me is read-only. It does not update your stories or project files. To add new material, run `/just-talk` in classic mode.

**Session duration** is set in `settings.yaml` under `test_me.session_duration`. Options: `15`, `30`, `45` (default: `30`).

**Question pool** is built from `interview-questions.md` plus any questions in `answers_interview_questions` across all project YAMLs. Classic mode generates new questions when filing stories, so the pool grows automatically over time.

**Round-robin state** is tracked in `career-stories/evaluations/state.yaml`. This file records which questions have been asked, when, and how many times — ensuring you get broad coverage across sessions.

---

## Customising the Hint Questions

Edit `interview-questions.md` to add, remove, or reorganise the prompts shown at the start of each session. The skill picks 3 at random, spread across categories.

---

## Project Folder Structure

The skill expects (and creates) this structure in your working directory:

```
career-stories/
├── <project-name>/
│   ├── <project-name>.yaml       # metadata: company, themes, interview questions
│   └── source-data/              # raw notes from just-talk sessions
├── just-talk-scratch.md          # live scratchpad during a session
└── just-talk-archive/            # timestamped archive of every session
    └── just-talk-YYYY-MM-DD-HHMM.md
```

Each project YAML looks like:

```yaml
company: "Acme Corp"
project: "Payment Platform Rebuild"
description: "Led a cross-functional effort to..."
key_themes:
  - stakeholder_alignment
  - platform_thinking
answers_interview_questions:
  - "Tell me about a time you had to build trust across teams"
source_data: "source-data/"
```

---

## License

MIT — do whatever you want with it.
