---
name: create-agent
description: Create a new Hermes agent as an isolated profile
version: 1.3.0
metadata:
  hermes:
    tags: [agents, profiles, orchestration]
    category: agents
    requires_toolsets: [terminal]
---

# Create Agent

Creates a new Hermes agent as its own profile, writes its SOUL.md, writes the skills and framework references from the brief into its `skills/` folder, verifies it, and records it in `~/agent-factory/registry.md`.

This skill is a hub. Load the matching file before you act:

| You need | Load |
|---|---|
| How to ask the user questions, the plan recap, the final message | `templates/interview-guide.md` |
| The internal brief you fill in (never shown to the user) | `templates/agent-brief.md` |
| The new agent's SOUL.md | `templates/agent-soul-template.md` |
| A skill for the new agent | `templates/skill-template.md` |
| A framework reference file | `templates/framework-reference-template.md` |
| CLI commands and flags | `references/cli-cheatsheet.md` |
| Anything not covered here | https://hermes-agent.nousresearch.com/docs/llms.txt |

## Scope and verification
This skill is a working guide, not the full Hermes manual. If a command or feature is not here, do not conclude it does not exist. Check `hermes --help`, `hermes <command> --help`, or the docs index above. Never answer "Hermes can't do that" from memory.

## When to Use
- The user wants a new, durable agent: a coder, researcher, content writer, reviewer, and so on.
- The agent needs its own identity, memory, skills or credentials.

Prefer the lightest option that works:
1. A skill added to an existing agent.
2. `delegate_task` for quick parallel subtasks. A subagent starts from a fresh conversation, so pass it all the context.
3. A spawned process (`hermes -p <name> chat -q "..."`, with `background=true` for long tasks) for a long autonomous mission with full tool access.
4. A new profile, only for a durable agent with its own identity, memory or credentials.

Scheduled work goes to cron, not a spawned process.

## What goes where
| Brief item | Written to |
|---|---|
| Role, tone, hard limits | the new agent's `SOUL.md` |
| Each task or skill the agent must perform | `skills/<category>/<skill-name>/SKILL.md` |
| Each framework | `references/<framework>.md` inside every skill that uses it |
| Output formats, checklists | `templates/` inside the skill that uses them |
| Sample pieces and finished examples | `examples/` inside the matching skill |
| Small source documents the user supplies | `references/` inside the matching skill, unchanged |
| Large sources (a style-guide PDF, a book) | the user runs `/learn <path>` in the new agent's chat; it builds a knowledge-base skill with one file per topic |

Do not put frameworks in SOUL.md. It loads every turn, so long material belongs in skills, which load on demand.

## Procedure
1. **Interview and brief.** Follow `templates/interview-guide.md` to ask the user a few plain-language questions, and fill in `templates/agent-brief.md` yourself from their answers. You need a name, a one-line role, the tasks (these become skills), and the frameworks with the task each supports. Infer the rest and state assumptions in one line.
2. **Preflight.** Run `hermes profile list` and `command -v <name>`. Stop and pick a different name if either shows a match, because each profile installs a command alias at `~/.local/bin/<name>` that would shadow an existing command. Use lowercase letters, digits and hyphens.
3. **Plan and approval.** Show the plain-language recap card from the interview guide: no file paths, commands or technical terms. Keep the draft SOUL.md and the skill tree ready and show them if the user asks. Wait for a clear yes.
4. **Create the profile and resolve its path.**
   `hermes profile create <name> --clone --description "<one-line role>"`
   Then resolve the directory without hardcoding `~/.hermes`. This agent runs inside its own profile, so strip that profile off `$HERMES_HOME`:
   `ROOT="${HERMES_HOME:-$HOME/.hermes}"; ROOT="${ROOT%/profiles/*}"; PROFILE_DIR="$ROOT/profiles/<name>"`
   Check that `$PROFILE_DIR` exists and matches the path `hermes profile show <name>` reports before writing anything. If it does not exist, stop and use the reported path. Never create the profile directory yourself: skills written outside the real profile are never loaded.
   `--clone` copies this agent's config, `.env`, SOUL.md, skills and memory files into the new profile. It does not copy sessions, cron jobs or messaging channels. Steps 5 to 7 replace or remove what should not carry over. For a fully blank profile, run it without `--clone` (or with `--no-skills` to skip bundled skills) and tell the user to run `<name> setup`.
5. **Write the SOUL.md.** Fill in `templates/agent-soul-template.md` and write it to `$PROFILE_DIR/SOUL.md`, overwriting the cloned one. Aim for roughly 3 to 5 sentences of role plus short rules and limits. Leave frameworks out.
6. **Scrub the clone.**
   - Delete `$PROFILE_DIR/memories/MEMORY.md` and `USER.md`; they hold this agent's notes and the user's profile, not the new agent's.
   - Find the cloned copy of this skill with `find "$PROFILE_DIR/skills" -type d -name create-agent` (its category folder differs between installs) and delete it unless the user asked for an agent that also creates agents.
   - Remove other cloned skills the new agent does not need. For unmodified bundled skills, `hermes -p <name> skills opt-out --remove` deletes them and stops future re-seeding; it asks the user to confirm first.
   Do this before step 7, so the agent's own skills are never caught by the cleanup.
7. **Write the agent's skills.** For each task in the brief:
   - Create `$PROFILE_DIR/skills/<category>/<skill-name>/SKILL.md` from `templates/skill-template.md`. One task per skill. Names start with a lowercase letter and use only lowercase letters, digits, hyphens and underscores. Check `hermes -p <name> skills list` first so the name does not duplicate an existing skill; two different skills with the same name in one directory cannot be loaded by that name.
   - The `description` must be 60 characters or fewer, say what the skill does and when to use it, and be wrapped in double quotes if it contains a colon.
   - For each framework, create `references/<framework>.md` from `templates/framework-reference-template.md` inside every skill that uses it, and link it from that skill's Procedure. If two frameworks are the same idea under two names, write one file and mention both names. A skill with more than two references gets a short routing table ("User wants… → load…") near the top.
   - If the user supplied their own definition or a document, use it as written. If a framework has several common definitions, ask the user which one applies before writing it. Do not guess. If you must write one from general knowledge, mark the file's first line `Source: general knowledge, not confirmed by the user`.
   - Write lessons, not logs: a pitfall is a general rule plus one clause of why, attached to the step it affects, with no dates, ticket numbers or chat quotes. Keep each `SKILL.md` body under about 24,000 characters; Hermes warns above that.
   - Skills from a published source go through `hermes -p <name> skills install <identifier>` (`--category` and `--name` are available). Hub installs are security-scanned and `--force` never overrides a dangerous verdict.
   - If the user wants several skills loaded together under one command, create a bundle: `hermes -p <name> bundles create <bundle-name> --skill <a> --skill <b> -d "<what it is for>"`. Check `--help`. The skills must already exist.
8. **Configure.** Set an explicit working directory: `hermes -p <name> config set terminal.cwd <absolute path>`. Set a model only if the user asked: `hermes -p <name> config set model.default <provider/model>`. Never hand-edit config.yaml and never touch `.env`.
9. **Verify.**
   - `hermes profile show <name>`
   - Every skill folder under `$PROFILE_DIR/skills/` has a `SKILL.md` that starts with frontmatter containing `name` and `description`, and every `references/` link points to a file that exists.
   - `hermes chat -p <name> --toolsets skills -q "In one sentence, what is your role, and which skills do you have?"` The answer should match the SOUL.md and name the skills you wrote. SOUL.md changes apply to new sessions only. Skills added while a conversation is open are announced to the agent on its next message.
10. **Register.** Append one row to `~/agent-factory/registry.md`: name, role, date, working directory, skills written, notes.
11. **Report.** Send the final message from the interview guide. Give `hermes -p <name> chat` as the launch command, and mention `<name> chat` only if its wrapper directory is on PATH. Tell the user what is left for them: API keys or `hermes -p <name> auth add <provider>`, messaging setup with `<name> setup` if wanted, any large source documents to load with `/learn`, and any definitions still marked unconfirmed.

## Pitfalls
- Do not use Hermes jargon with the user (profile, SOUL.md, skill, clone, terminal.cwd) unless they use it first, and do not ask questions they cannot answer, such as which model or where a config file lives.
- `--clone` copies memory and skills from the creating agent. Skipping step 6 leaks this agent's notes and its create-agent skill into the child.
- Writing skills before the scrub risks deleting them. Keep the order: scrub, then write.
- A vague skill description means the agent will not pick the skill. Name the task and the trigger.
- Do not invent framework definitions. Ambiguous ones (for example "5-5-5" content mixes) need the user's version.
- Do not summarize a large source document yourself. Point the user to `/learn`, which keeps one reference file per topic instead of a lossy summary.
- Do not hardcode `~/.hermes`. Inside a profile, `$HERMES_HOME` already points at that profile, so profile paths must be built from the root as in step 4.
- `skills.write_approval` gates the `skill_manage` tool. Files you write with terminal tools are not staged by it, so the user's approval at step 3 is the review.
- Anthropic, OpenAI Codex and xAI OAuth logins are shared from the root, not copied. A profile needing its own login must run `hermes -p <name> auth add <provider>`.
- Messaging bot tokens are never cloned, on purpose: a bot can belong to only one profile.
- A profile is not a sandbox. `terminal.cwd` only sets where commands start.
- Profile deletion is destructive and asks the user to type the name. Only run it when the user explicitly asks.

## Verification
- `hermes profile list` shows the new profile.
- The skill tree matches what the user approved in step 3.
- The one-shot chat names the new skills.
- `$PROFILE_DIR/memories/` holds no copied MEMORY.md or USER.md.
- `find "$PROFILE_DIR/skills" -type d -name create-agent` returns nothing (unless the child was meant to create agents).
- The registry has a new row.
