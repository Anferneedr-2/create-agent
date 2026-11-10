# Hermes commands used by create-agent

Taken from the official docs (Profiles, Skills System, Bot Mode) and the official hermes-agent skill. If a flag is rejected, run `<command> --help`, and check https://hermes-agent.nousresearch.com/docs/llms.txt for anything not listed here.

## Profiles
| Goal | Command |
|---|---|
| List profiles | `hermes profile list` |
| Show one | `hermes profile show <name>` |
| Blank profile | `hermes profile create <name> --description "<role>"` |
| Clone config, SOUL, skills, MEMORY/USER | `hermes profile create <name> --clone --description "<role>"` |
| Clone from a named profile | `hermes profile create <name> --clone-from <source>` |
| Clone everything except sessions and cron | `hermes profile create <name> --clone-all` |
| Skip bundled skills | `hermes profile create <name> --no-skills` |
| Set or regenerate description | `hermes profile describe` |
| Set a config value | `hermes -p <name> config set <key> <value>` |
| One-shot chat | `hermes chat -p <name> -q "<prompt>"` |
| Ask what skills it has | `hermes chat -p <name> --toolsets skills -q "What skills do you have?"` |
| Interactive chat | `<name> chat` |
| Per-profile login | `hermes -p <name> auth add <provider>` |
| Configure keys, model, gateway | `<name> setup` |
| Rename | `hermes profile rename <old> <new>` |
| Export (keys stripped) | `hermes profile export <name>` |
| Delete (user must confirm) | `hermes profile delete <name>` |

## Skills
| Goal | Command |
|---|---|
| List a profile's skills | `hermes -p <name> skills list` |
| Search hubs | `hermes skills search <query>` |
| Inspect before installing | `hermes skills inspect <identifier>` |
| Install | `hermes -p <name> skills install <identifier> [--category X] [--name N]` |
| Install from a URL | `hermes skills install https://example.com/SKILL.md --name <slug>` |
| Stop bundled seeding | `hermes -p <name> skills opt-out` |
| Also delete unmodified bundled skills | `hermes -p <name> skills opt-out --remove` (asks first) |
| Lock a skill against agent edits | `hermes curator lock <skill>` |
| Group skills under one command | `hermes -p <name> bundles create <bundle> --skill <a> --skill <b> -d "<desc>"` |
| List bundles | `hermes bundles list` |

Inside a chat: `/skills pending`, `/skills approve <id>` (when `skills.write_approval` is on), `/bundles`, and `/learn <path or URL>` to turn a source into a skill.

## Layout
`$HERMES_HOME/skills/<category>/<skill>/` holds `SKILL.md` (required) and optional `references/`, `templates/`, `scripts/`, `examples/`, `assets/`. A profile lives at `<root>/profiles/<name>/` with `config.yaml`, `.env`, `SOUL.md`, `memories/`, `skills/`, `cron/`, `sessions/`.

## Rules from the docs
- Settings go in config.yaml (set with `hermes config set`), secrets only in `.env`.
- Messaging channels (bot tokens, allowlists) are not cloned unless `--clone-channels` is passed.
- OAuth logins for Anthropic, OpenAI Codex and xAI are shared from root, not copied.
- Profiles are not sandboxes. Set `terminal.cwd` for a predictable start directory.
- Skill names start with a lowercase letter; descriptions are 60 characters or fewer.
- `--description` helps the Kanban orchestrator route work to the profile.
