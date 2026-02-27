# Just Talk — Setup Instructions

This folder contains the `just-talk/` skill. Pick your AI coding tool below and follow the setup steps.

---

## Claude Code ⭐ Recommended

Claude Code is the primary supported platform. The skill uses Claude Code's native skill system and slash command interface.

### Step 1: Add the skill

Copy the `just-talk/` folder into your project's `.claude/skills/` directory:

```
your-project/
└── .claude/
    └── skills/
        └── just-talk/
            ├── SKILL.md
            ├── settings.yaml
            └── interview-questions.md
```

### Step 2: Add the slash command

Create a file at `.claude/commands/just-talk.md` with the following content:

```markdown
Invoke the just-talk skill and run a career story capture session.
```

This registers `/just-talk` as a slash command in Claude Code chat.

### Step 3: Run it

In Claude Code, type:

```
/just-talk
```

The first time you run it, a setup wizard will walk you through your preferences. From then on, it goes straight into the conversation.

### Notes
- The `career-stories/` folder (where stories are filed) should be your working directory when you run the skill
- You can change any setting at any time by editing `just-talk/settings.yaml` directly
- To re-run the setup wizard, set `first_run: true` in `settings.yaml`

---

## Cursor

Cursor doesn't have a native skill system, but you can load Just Talk as a project rule that's always available.

### Step 1: Add the rule

Copy the contents of `just-talk/SKILL.md` into a new file at:

```
your-project/
└── .cursor/
    └── rules/
        └── just-talk.mdc
```

### Step 2: Add front matter

Cursor rules use `.mdc` format. Add this to the top of the file before the skill content:

```
---
description: Just Talk — career story capture session
globs:
alwaysApply: false
---
```

### Step 3: Run it

In Cursor chat, reference the rule explicitly:

```
Use the just-talk rule to start a career story session.
```

### Notes
- Cursor doesn't support `settings.yaml` natively — hardcode your preferred tone and mode directly in the `.mdc` file by editing the relevant sections
- There's no automatic first-run setup; configure preferences manually in the rule file before your first session

---

## ChatGPT

ChatGPT doesn't have a file-based skill system, but you can run Just Talk as a custom GPT or via a system prompt in a Project.

### Option A: Custom GPT

1. Go to [chatgpt.com/gpts/editor](https://chatgpt.com/gpts/editor)
2. Under **Instructions**, paste the full contents of `just-talk/SKILL.md`
3. Name it "Just Talk" and save
4. Start a new chat with your custom GPT whenever you want a session

### Option B: ChatGPT Project (recommended for regular use)

1. Create a new Project in ChatGPT
2. Go to **Project instructions** and paste the full contents of `just-talk/SKILL.md`
3. Any chat in that project will have Just Talk available

### Notes
- ChatGPT can't read or write files, so the filing phase (Phase 2) won't work automatically — copy the session summary manually into your project folders
- Hardcode your preferred tone in the instructions by editing the `tone` value in the pasted skill content
- The status line, archive, and YAML update features require a file-capable environment (Claude Code or local MCP tools)

---

## Gemini / Google AI Studio

> **Note:** These instructions are written for Google AI Studio and Gemini. If you're using a different Google AI product, the setup will be similar — adapt the steps to wherever your tool stores system instructions or custom prompts.

### Option A: Google AI Studio (System Prompt)

1. Go to [aistudio.google.com](https://aistudio.google.com)
2. Create a new prompt or open an existing one
3. In the **System instructions** field, paste the full contents of `just-talk/SKILL.md`
4. Save as a reusable prompt

### Option B: Gemini in Google Workspace

1. Open Gemini in Docs, Gmail, or your Workspace app
2. Use the **Gems** feature (if available in your plan) to create a custom Gem
3. Paste `just-talk/SKILL.md` as the Gem's instructions

### Notes
- File read/write capabilities depend on which Google product and plan you're using
- The filing phase (Phase 2) may need to be done manually if the tool doesn't have access to your local filesystem
- Hardcode your preferred tone and mode in the pasted instructions

---

## Universal Fallback (Any AI Chat Tool)

If your tool supports custom system instructions or a persistent prompt, paste the contents of `just-talk/SKILL.md` there. The skill is designed to work as a pure conversational prompt — the filing and archiving phases work best with file access, but the listening and questioning phases work anywhere.
