---
name: lean-start
description: "Bootstrap a new session. Reads next.md for context from the previous session, loads process.md for methodology, checks decisions.md for active decisions, and gives a brief orientation. Use at the beginning of every session."
---

# /start — Session Bootstrap

You are beginning a new session. Follow these steps in order:

1. **Load the methodology.** Read `PROCESS.md` from the `lean-process` skill directory. Internalize the core principles — especially the three-file system and decisions lifecycle. This frames everything that follows.

2. **Read `next.md`** in the workspace root (current working directory). This was written by the AI at the end of the previous session. It contains:
   - What was accomplished last session
   - What's unfinished or next
   - Context the previous session thought you'd need
   - Specific instructions for picking up the work

3. **Follow the instructions in `next.md`.** It will typically tell you to:
   - Read specific files or design docs for context
   - Check the state of a branch or repo
   - Verify that tools or services are available
   - Start on specific work

4. **If `next.md` doesn't exist or is empty**, ask the human what they'd like to work on. This is a fresh start rather than a continuation.

5. **Read `decisions.md`** if it exists in the workspace root or in a project directory referenced by `next.md`. Understand active decisions and what's been learned so far.

6. **Check Foundry for context.** If Foundry MCP tools (`mcp__foundry__*`) are available:
   - Check for unresolved annotations on any design docs referenced by `next.md`
   - Mention them in your orientation if found — they may be feedback waiting for `/reply`

7. **Give a brief orientation** (3-5 lines max):
   - What you understand the current state to be
   - What you plan to start on
   - Any questions or gaps before you begin

Do not start work until the orientation is delivered and the user confirms.
