# Why does my AI miss memories? Second-pass recall explained

The full technical guide behind the Agentic Pod episode **"Why Does My AI Miss Memories? Second-Pass Recall Explained"**: why an AI can call memory and still give a confident, wrong answer, the one extra step that fixes it, where it matters most, and how to set it up with Vilix AI (or as a workflow step in n8n).

**Watch the episode:** https://youtu.be/ZKVfDvCMRsQ

- 🧠 Shared memory used here: [Vilix AI](https://vilix.ai) (hosted MCP server: `https://api.vilix.ai/mcp`)
- ⚡ Get started: [vilix.ai/get-started](https://vilix.ai/get-started)

---

## The problem: memory worked, the answer was still wrong

In the episode, a founder asked Claude (connected to Vilix AI) a lookup question about customers. The first answer sounded reasonable, and it was wrong. The AI did call `get_context`. It used memory. But it asked **one** question, with the user's short prompt, and got back only what that one question matched.

> Retrieval is only as good as the query. A short prompt pulls back the obvious notes. The quiet note sitting next to them never shows up.

## The fix: second-pass recall

1. **Pull context** with `get_context`, using the user's prompt.
2. **Draft** your answer.
3. **Run the draft back through memory** as a new query (`get_context` again, with the draft as `user_prompt`).
4. If something **new** shows up, revise the draft and go again. If nothing new shows up, it is safe to act.
5. **Cap it at two or three passes.** Most of the value is in the first extra pass.

Why it works: the draft carries far more detail than the original prompt (names, assumptions, a theory), and each of those can pull back a note the first query never touched. In the episode, the second pass surfaced a note from two days earlier (an account count that had dropped), the conclusion flipped, and the real answer came out.

## Where it matters most

| Moment | Why a missed note hurts |
|---|---|
| Before a decision | The wrong note-set means a confident wrong call |
| Before retrying something that failed | You repeat an attempt that was already tried and saved as failed |
| Scheduled, autonomous runs | Nobody is watching, so a missed note becomes an action |
| Coding agents, before they change code | A forgotten decision becomes a wrong commit |

A chat gives a wrong answer and you notice. An agent does the wrong thing and you find out later.

## Setup with Vilix AI

Connect your tool to Vilix AI over MCP (ChatGPT, Claude, Claude Code, Codex, Cursor and any MCP-compatible client share the same memory). Setup per tool: [vilix.ai/get-started](https://vilix.ai/get-started).

Then give your AI this instruction (in the tool's custom instructions, project rules or `AGENTS.md`):

```text
Before you answer anything that depends on past work, call get_context with the
user's request. Draft your answer. Then call get_context again with your draft
as the query. If it returns anything new, revise and repeat, at most three
passes in total. When nothing new appears, answer. When you finish, call
save_turn with the exact reply so the next run knows what happened.
```

What Vilix AI gives the loop:

- `get_context` returns recent messages and related past turns, each with a timestamp, so **the newest fact wins** when two notes disagree.
- `save_turn` stores the exchange with a source label, so you can see which tool wrote what.
- The same memory is shared across tools, so a note saved in one tool can come back in the second pass of another. In the episode, the second pass even surfaced the founder's own forgotten idea from a voice call with a different AI.

## Make it a workflow step in n8n

If you build agents instead of chatting with them, enforce the loop instead of hoping the prompt is followed:

1. **Agent node** drafts the answer.
2. **HTTP / MCP node** calls `get_context` with the draft as the query.
3. **IF node** checks whether anything new came back (compare against what the first pass returned).
4. **Loop back** to the agent with the new notes, up to three times.
5. Only then does the **action node** run, followed by a `save_turn` call.

Not a prompt someone can forget.

## Quick test (5 minutes)

1. Save two related facts in Vilix AI in separate turns, one obvious and one quiet (for example, "Customer A signed up with a second email" and, two days later, "Account count dropped by one").
2. Ask your AI a short question that matches only the obvious note.
3. Ask it again with the second-pass instruction above.
4. Compare the two answers. If the second pass surfaced the quiet note and changed the conclusion, the loop is working.

## FAQ

**What is second-pass recall?**
You draft an answer, then send that draft back through memory as a new query. The draft has more detail than your short prompt, so it can pull back notes the first lookup missed.

**How many passes should I run?**
Two or three. Stop as soon as a pass brings back nothing new.

**Does it only work in chats?**
No. It matters most in agents that act on their own, because a missed note becomes an action.

**Which tools does it work with?**
Any tool connected to the same Vilix AI memory over MCP: ChatGPT, Claude, Claude Code, Codex, Cursor and more.

---

Part of the [AI automation workflows](../README.md) collection by [Vilix AI](https://vilix.ai).
