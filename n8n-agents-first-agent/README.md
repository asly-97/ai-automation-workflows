# How do you build your first n8n agent?

A practical first-day guide to n8n Agents: what they are, how to build one step by step, how to run it from Slack or on a schedule, what it costs, and what is missing in the preview. Companion to the Agentic Pod episode "How Do You Build Your First n8n Agent?".

- 🎧 Watch the episode: [Agentic Pod on YouTube](https://youtu.be/TL3RHq6SxNo)

Facts in this guide come from the official n8n documentation and the n8n community launch post, checked on 2026-10-16. Agents are a preview feature, so check the docs before relying on a detail.

## What n8n Agents are

- A new kind of object that lives in its own **Agents** tab, next to workflows. It is separate from the existing AI Agent node, which keeps working and is not deprecated.
- You define an agent once: instructions, a model and tools. It decides its own next steps within the goal you set.
- You can chat with it, use it as a node in a workflow, connect it to a channel, or run it on a schedule.
- Your existing workflows can be its tools, so the agent chooses which one to run, and when.
- Availability: preview from n8n version 2.32.3. On Cloud it is on all plans. Self-hosted is supported from the same version, with a few extra requirements (below).

## Build your first agent

1. Open your project, go to the **Agents** tab and select **Create Agent**. Or describe the agent to the AI Assistant and let it draft the name, instructions and tools for you to refine.
2. **Name and model.** Pick a provider and a model, and add a credential if n8n asks for one.
3. **Instructions.** Write the role, the tone, the response format, the constraints and which tools to prefer. If the agent misbehaves, fix the instructions before you add more tools.
4. **Tools.** Choose built-in n8n integrations, workflows from the same project, custom tools defined with a JSON schema, or tools from an MCP server. Attach credentials. Mark sensitive tools as **requiring approval**.
5. **Optional building blocks.** Web search (the model's own, or Brave Search or a self-hosted SearXNG), skills (instructions bundled with the tools for one job), knowledge files (csv, pdf, markdown, txt), and sub-agents you describe a "when to call" rule for.
6. **Preview.** Test the draft in the chat panel. If Preview is disabled, fix the configuration errors it lists.
7. **Publish.** Publishing snapshots the draft as the live version. Channels and schedules only run the published version, so you can keep editing the draft safely. **Unpublish** takes the agent offline and keeps the draft. Each publish is a version you can restore.

## Run it where you work

- **Channels:** Slack, Telegram and Linear, each with its own connection and permission setup.
- **Schedules:** hourly, daily, weekly, monthly or a custom cron expression. Each schedule shows its next run, last run and last outcome.
- **Inside workflows:** the **Message an Agent** node sends a message to an agent and returns the reply to the next node. It can stream replies, reuse sessions and constrain the output to a JSON schema. Use **Message an Agent Tool** on an AI Agent node so that agent can hand work to another agent.
- **Approvals:** when an agent reaches a tool marked as requiring approval, it pauses until you press **Approve** or **Reject**, then continues from that point.
- **Sessions:** every conversation is saved with its messages, the tools it used and pending approvals, in the **Sessions** tab.

## What it costs

- One **turn** (one message and the agent's reply) counts as **one execution**. Workflow-tool calls and sub-agent calls inside that turn are not counted separately.
- Agent turns share your normal workflow execution quota.
- Agents run on your own model credential, so token costs go straight to your model provider. Watch both meters.

## Memory

- By default an agent keeps the context of the current conversation.
- To remember across conversations, turn on **episodic memory** in the Memory tab. It needs an OpenAI credential.

## Gotchas and limits (preview)

- Agents can make mistakes and their behavior may change, so test in Preview and use approvals for anything that sends email, changes data or replies externally.
- Not available yet: agents that continuously watch an event stream, full multi-agent orchestration, and calling an agent from outside n8n (reach it through the chat, a channel, a schedule or a workflow).
- Queue mode is not supported for agents yet. Run agents in regular mode.
- Self-hosted: channels need a public `WEBHOOK_URL`, and knowledge bases need a configured sandbox. Connecting channels such as Telegram can fail on self-hosted instances.
- Admins can switch agents on or off under **Settings > Agents**. Turning them off does not affect the AI Agent node.

## Who it is for

- Good fit: anyone already building in n8n who wants one agent that can use their workflows, answer in Slack and run on a schedule.
- Start small: pick one low-risk job, for example a Slack helper that summarizes something, and keep approvals on.
- Wait if you need queue mode, or an agent that other systems can call.

## First-day checklist

1. Update n8n to 2.32.3 or later (the community post recommends 2.34.x).
2. Create one agent with short, specific instructions and one tool.
3. Preview it with five real prompts before you publish.
4. Connect one channel or one schedule, not both.
5. Mark every tool that sends or changes something as requiring approval.
6. After a week, check your execution count and your model provider's bill.

## Sources

- Build and manage agents (n8n docs): https://docs.n8n.io/build/build-and-manage-agents
- Introducing n8n Agents (n8n community): https://community.n8n.io/t/introducing-n8n-agents-a-new-way-to-build-agents-you-set-up-once-and-use-anywhere/306323
