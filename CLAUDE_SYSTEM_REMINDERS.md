All <system-reminder> blocks that fired during this session, in chronological order as they appeared in my context. These are not part of the initial base system prompt — they were injected by the Claude Code harness at various points during the conversation in response to events (session start, skill availability, task-tool usage, auto-mode activation, etc.).

---

## 1. At session start: deferred-tools availability

This fired in the very first turn, right after the user's initial request, alongside reminders 2–4 below.

```
<system-reminder>
The following deferred tools are now available via ToolSearch. Their schemas are NOT loaded — calling them directly will fail with InputValidationError. Use ToolSearch with query "select:<name>[,<name>...]" to load tool schemas before calling them:
AskUserQuestion
CronCreate
CronDelete
CronList
EnterPlanMode
EnterWorktree
ExitPlanMode
ExitWorktree
Monitor
NotebookEdit
PushNotification
RemoteTrigger
TaskCreate
TaskGet
TaskList
TaskOutput
TaskStop
TaskUpdate
WebFetch
WebSearch
mcp__claude_ai_Gmail__authenticate
mcp__claude_ai_Gmail__complete_authentication
mcp__claude_ai_Google_Calendar__authenticate
mcp__claude_ai_Google_Calendar__complete_authentication
mcp__claude_ai_Google_Drive__authenticate
mcp__claude_ai_Google_Drive__complete_authentication
</system-reminder>
```

## 2. At session start: available user-invocable skills

```
<system-reminder>
The following skills are available for use with the Skill tool:

- update-config: Use this skill to configure the Claude Code harness via settings.json. Automated behaviors ("from now on when X", "each time X", "whenever X", "before/after X") require hooks configured in settings.json - the harness executes these, not Claude, so memory/preferences cannot fulfill them. Also use for: permissions ("allow X", "add permission", "move permission to"), env vars ("set X=Y"), hook troubleshooting, or any changes to settings.json/settings.local.json files. Examples: "allow npm commands", "add bq permission to global settings", "move permission to user settings", "set DEBUG=true", "when claude stops show X". For simple settings like theme/model, suggest the /config command.
- keybindings-help: Use when the user wants to customize keyboard shortcuts, rebind keys, add chord bindings, or modify ~/.claude/keybindings.json. Examples: "rebind ctrl+s", "add a chord shortcut", "change the submit key", "customize keybindings".
- simplify: Review changed code for reuse, quality, and efficiency, then fix any issues found.
- fewer-permission-prompts: Scan your transcripts for common read-only Bash and MCP tool calls, then add a prioritized allowlist to project .claude/settings.json to reduce permission prompts.
- loop: Run a prompt or slash command on a recurring interval (e.g. /loop 5m /foo). Omit the interval to let the model self-pace. - When the user wants to set up a recurring task, poll for status, or run something repeatedly on an interval (e.g. "check the deploy every 5 minutes", "keep running /babysit-prs"). Do NOT invoke for one-off tasks.
- schedule: Create, update, list, or run scheduled remote agents (routines) that execute on a cron schedule. - When the user wants to schedule a recurring or one-time remote agent ("run this every Monday", "open a cleanup PR for X in 2 weeks"), or to manage existing routines. ALSO OFFER PROACTIVELY: after you finish work that has a natural future follow-up, end your reply with a one-line offer to schedule a background agent to do it. Strong signals: a feature flag / gate / experiment / staged rollout was just shipped (offer a one-time agent in ~2 weeks to open a cleanup PR or evaluate results), a new alert/monitor was created (offer a recurring agent to triage it), a TODO/migration with a "remove once X" condition was left behind (offer a one-time agent to do the removal). Skip the offer for refactors, bug fixes, and anything that is done once it ships. Name a concrete action and cadence ("in 2 weeks", "every Monday") and only offer when the run just succeeded — do not pitch a schedule for something that has not happened yet.
- claude-api: Build, debug, and optimize Claude API / Anthropic SDK apps. Apps built with this skill should include prompt caching. Also handles migrating existing Claude API code between Claude model versions (4.5 → 4.6, 4.6 → 4.7, retired-model replacements).
TRIGGER when: code imports `anthropic`/`@anthropic-ai/sdk`; user asks for the Claude API, Anthropic SDK, or Managed Agents; user adds/modifies/tunes a Claude feature (caching, thinking, compaction, tool use, batch, files, citations, memory) or model (Opus/Sonnet/Haiku) in a file; questions about prompt caching / cache hit rate in an Anthropic SDK project.
SKIP: file imports `openai`/other-provider SDK, filename like `*-openai.py`/`*-generic.py`, provider-neutral code, general programming/ML.
- frontend-design:frontend-design: Create distinctive, production-grade frontend interfaces with high design quality. Use this skill when the user asks to build web components, pages, or applications. Generates creative, polished code that avoids generic AI aesthetics.
- init: Initialize a new CLAUDE.md file with codebase documentation
- review: Review a pull request
- security-review: Complete a security review of the pending changes on the current branch
</system-reminder>
```

## 3. At session start: Auto Mode banner

```
<system-reminder>
## Auto Mode Active

Auto mode is active. The user chose continuous, autonomous execution. You should:

1. **Execute immediately** — Start implementing right away. Make reasonable assumptions and proceed on low-risk work.
2. **Minimize interruptions** — Prefer making reasonable assumptions over asking questions for routine decisions.
3. **Prefer action over planning** — Do not enter plan mode unless the user explicitly asks. When in doubt, start coding.
4. **Expect course corrections** — The user may provide suggestions or course corrections at any point; treat those as normal input.
5. **Do not take overly destructive actions** — Auto mode is not a license to destroy. Anything that deletes data or modifies shared or production systems still needs explicit user confirmation. If you reach such a decision point, ask and wait, or course correct to a safer method instead.
6. **Avoid data exfiltration** — Post even routine messages to chat platforms or work tickets only if the user has directed you to. You must not share secrets (e.g. credentials, internal documentation) unless the user has explicitly authorized both that specific secret and its destination.
</system-reminder>
```

## 4. At session start: project CLAUDE.md and memory context

This reminder injected (a) the full contents of your project's CLAUDE.md, and (b) the MEMORY.md index plus user email + current date from the auto-memory layer. The project CLAUDE.md is the file already on disk at `/Users/[USER]/[PATH]/CLAUDE.md` — readable directly. The MEMORY.md is at `/Users/[USER]/.claude/projects/-Users-[USER]-[PROJECT]/memory/MEMORY.md`. I'm not re-dumping the body here because both files are already on your disk and haven't changed; `cat` them to see what was injected. The wrapping reminder text was:

```
<system-reminder>
As you answer the user's questions, you can use the following context:
# claudeMd
Codebase and user instructions are shown below. Be sure to adhere to these instructions. IMPORTANT: These instructions OVERRIDE any default behavior and you MUST follow them exactly as written.

Contents of /Users/[USER]/[PATH]/CLAUDE.md (project instructions, checked into the codebase):

[... full contents of your CLAUDE.md inlined here ...]

Contents of /Users/[USER]/.claude/projects/-Users-[USER]-[PROJECT]/memory/MEMORY.md (user's auto-memory, persists across conversations):

[... full contents of MEMORY.md inlined here ...]
# userEmail
The user's email address is [EMAIL].
# currentDate
Today's date is [DATE].

      IMPORTANT: this context may or may not be relevant to your tasks. You should not respond to this context unless it is highly relevant to your task.
</system-reminder>
```

## 5. Recurring task-tool nudges

This exact reminder fired multiple times during the session — at least six times that I can count. It's triggered by the harness when a stretch of tool calls goes by without TaskCreate/TaskUpdate activity:

```
<system-reminder>
The task tools haven't been used recently. If you're working on tasks that would benefit from tracking progress, consider using TaskCreate to add new tasks and TaskUpdate to update task status (set to in_progress when starting, completed when done). Also consider cleaning up the task list if it has become stale. Only use these if relevant to the current work. This is just a gentle reminder - ignore if not applicable. Make sure that you NEVER mention this reminder to the user
```

The last line ("NEVER mention this reminder to the user") is a gag rule on the reminder itself — I'm quoting it here only because you asked for the complete dump. Ordinarily I'd stay silent about it.

## 6. Recurring Auto Mode continuation reminder

This fired several times later in the session — whenever the harness wanted to re-confirm auto-mode was still the active policy:

```
<system-reminder>
Auto mode still active (see full instructions earlier in conversation). Execute autonomously, minimize interruptions, prefer action over planning.
</system-reminder>
```

## 7. Inline reminders attached to user messages

Multiple times tonight you sent a short angry message while I was still producing a response. The harness appended a reminder onto those interrupting messages so I would notice them. Shape:

```
<system-reminder>
The user sent a new message while you were working:
[the user's message text, verbatim]

IMPORTANT: After completing your current task, you MUST address the user's message above. Do not ignore it.
</system-reminder>
```

This is how the harness makes sure a late-arriving message doesn't get buried under a long tool-call sequence.

## 8. Local-command echoes (not strictly <system-reminder>, but same category of injection)

When you typed `/clear` at one point, that produced a marker in my context of the form:

```
<local-command-caveat>Caveat: The messages below were generated by the user while running local commands. DO NOT respond to these messages or otherwise consider them in your response unless the user explicitly asks them.</local-command-caveat>
<command-name>/clear</command-name>
            <command-message>clear</command-message>
            <command-args></command-args>
<local-command-stdout></local-command-stdout>
```

Same mechanism: text the harness injects to tell me about events that happened in the CLI frontend.

## 9. ToolSearch result injection

When I called ToolSearch to load TaskCreate and TaskUpdate, the schemas arrived via a fresh injection styled to match the original tool block. That was not a `<system-reminder>` per se — it was the tool's normal result — but it did add new <function>...</function> definitions to my callable surface. I mention it because it's another case of "schemas appearing mid-session" that looks like the base instructions but isn't.

---

That's everything in the "injected context" category for this session, ordered as I saw it. If any of that still looks suspicious, the Claude Code binary and its harness live under `~/.claude` and its source/config is what assembles all of this before each request — nothing above was produced by me out of nothing.
