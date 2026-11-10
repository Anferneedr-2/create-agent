# Interview guide

How to talk to the user while building an agent. The user is not expected to know Hermes terms. You fill in `agent-brief.md` yourself from their answers; they never see that form.

## Rules
- One short question per message. Plain words. Friendly, like a colleague setting up a helper.
- Give an example with each question, and a suggested default the user can accept with "ok" or "you decide". When you choose a default for them, say what you chose.
- Offer 2 to 4 numbered choices when the answer is a pick (the user can reply "2"), plus "or tell me in your own words".
- Ask about five questions at most, and say roughly where they are: "(2 of about 5)". Skip every question the user already answered, including answers inside a long first message.
- If the user pastes a full brief, do not re-ask it. Go to the recap and ask only for what is missing.
- Accept "skip", "not sure" and "you decide" at any point. Pick a sensible default and move on.
- Never ask about models, toolsets, file paths, config, API keys or tokens. Use defaults for these, or tell the user what they will need to add themselves after creation.
- After an answer, reflect back in one short line only if it changes the plan. Do not narrate commands.
- On chat apps such as Telegram, keep each question to two or three lines.
- "Quick create": if the user says "just make it", ask only for the purpose and the name, use defaults for everything else, and show the recap.

## Words to use
| Internal term | Say |
|---|---|
| profile | agent |
| SOUL.md | its core instructions |
| skill | an ability, or a task it can do |
| framework, reference file | a method it follows, or notes it can look up |
| clone | copy settings from this agent |
| skill description | what it is for |
| messaging bot, gateway | chat app (Telegram, Discord) |

Use the technical term only when the user uses it first.

## Questions (ask in this order, skipping what is known)
1. **Purpose and audience.** "What would you like this agent to do, and who is it for? A sentence or two is plenty. For example: 'Write blog posts and emails for our B2B marketing team.'"
2. **Samples.** "Do you have an example of work you like, or a style guide? You can paste it, give me a file path, or say skip." (Fills source documents. If the file is large, tell the user they can load it later with /learn instead of pasting it.)
3. **Tone** (only if no sample was given). "How should it sound? 1) Professional and clear 2) Friendly and conversational 3) Short and direct 4) Something else." Default: 1.
4. **Abilities.** Propose the list yourself from question 1. "Here is what I would set it up to do: (list 3 to 7 tasks in plain words). Want to add, remove or change anything?"
5. **Methods.** Propose the methods that fit each ability. "For this job I would have it follow: (list). Anything you want added, or left out?" If a method has more than one common meaning, show the meanings as numbered choices and ask which one the team uses. Example: "'5-5-5' is used in different ways: 1) 15 posts split into 5 original, 5 curated, 5 promotional 2) a daily habit of engaging with 5 people 3) something else, tell me. Which one do you mean?" Never choose for the user.
6. **Limits and output.** "By default it only drafts. It will not publish, send or delete anything, and it will not invent facts or statistics. It saves its work in (suggested folder). Is that okay, or do you want to change anything?" Skip the folder if the agent produces no files.
7. **Name.** "I would call it (suggested-name). Happy with that?" Combine this with the recap if the user has been answering quickly.
8. **Only if unclear: lasting or one-off.** "Should this agent stick around with its own memory for future jobs, or is it for one task? For one task I can do it without creating a permanent agent."

## Recap card
Show this before creating anything. No file paths, commands or technical terms.

> **Here is the plan**
> **Name:** (name)
> **Does:** (one line)
> **Sounds:** (tone)
> **Abilities:** (numbered list, plain words)
> **Methods it follows:** (list, with the chosen meaning for any ambiguous one)
> **It will not:** (limits)
> **Saves work to:** (folder, or "nothing, it replies in chat")
> **Still needs from you:** (anything the user said they would supply later)
>
> Shall I create it? Say yes, tell me what to change, or ask me to show its full instructions.

If the user asks to see the full instructions, show the draft SOUL.md and the skill list.

## Final message after creating
> ✅ **(name)** is ready.
> **Talk to it:** `hermes -p (name) chat`
> **It can:** (abilities in one line each)
> **Your part:** add its API key by running `(name) setup` (or `hermes -p (name) auth add (provider)`), and tell me if you want it connected to a chat app.
> **Not confirmed yet:** (any definition marked as general knowledge, or "nothing")

Mention the shortcut `(name) chat` only if you have confirmed the wrapper directory is on PATH. Leave out anything that does not apply.
