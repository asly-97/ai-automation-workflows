# How do you give a WhatsApp AI agent memory between conversations?

The practical guide behind the Agentic Pod episode **"How Do You Give a WhatsApp AI Agent Memory Between Conversations?"**: why WhatsApp agents forget, the four things they should remember, the load and save pattern, and how Vilix AI works as the shared memory store.

## Why a WhatsApp agent forgets everything

Three layers all start blank, and none of them is broken:

1. **The webhook.** WhatsApp delivers each inbound message to your server as its own webhook event. Your handler wakes up, receives one message, and exits. There is no running conversation process on your side.
2. **The model call is stateless.** A chat completion only contains the messages you put in that request. Send only the new message and the model cannot know the same customer answered your qualification questions yesterday.
3. **History stays in the tool.** Most setups keep history inside the chat platform or the automation tool, if they keep it at all. When the conversation ends, that history is not loaded into the next conversation.

Result: a returning customer gets asked for their name, their order number, and their problem all over again.

## What a WhatsApp agent should remember

1. **Identity facts:** the customer's name, language and account details, keyed by phone number.
2. **Decisions already made:** a refund already offered, a human already taking over, a discount already approved.
3. **Open loops:** the question the customer was last asked, the document they still owe you, the appointment still to confirm.
4. **The full conversation history:** what was actually said in earlier conversations, so a later run can revisit the real exchange instead of a shortened note about it.

Items 1 to 3 stop repeat questions. Item 4 stops contradictions.

## The pattern, step by step

1. **Key everything by phone number.** The customer's number is the stable ID in every webhook payload.
2. **Save each turn as it happens.** Store the inbound message and your agent's reply, in order.
3. **Deduplicate webhook retries.** Store the message ID with each inbound turn and skip a turn you have already saved, or the agent answers twice and remembers the exchange twice.
4. **Load memory before the model reasons.** At the start of each turn, retrieve the history and facts relevant to this customer and this message, and put them in the model call.
5. **Save the outcome, not just the reply.** After the agent acts, write back what it decided and what is still open.
6. **Repeat the same load and save in every tool.** A human teammate, a scheduled follow-up agent and the WhatsApp agent need to read and write the same memory, or each keeps a private version of the customer.

## Where Vilix AI fits

Vilix AI is a shared memory layer your agent connects to instead of a store you build and maintain yourself. It is cloud-hosted, so there is zero infrastructure to deploy or manage next to your webhook server.

- Before the model reasons, the agent calls **get_context** and loads the relevant saved context for that customer.
- After it replies, it calls **save_turn** and saves the full exchange back.
- Retrieval is semantic and keyword-based at the same time, so recall finds what a conversation meant and still matches exact strings such as order IDs literally.
- Vilix AI stores full conversation history, not just extracted facts.
- When two tools save conflicting information, the most recently saved version is the one retrieval returns, and every save carries a source label.
- **Agencies:** give each client their own Vilix AI account. Every agent built for that client connects to that client's account, so the memory belongs to the client's business and never mixes with another client's.

A headless agent connects with an API key as a Bearer token to `https://api.vilix.ai/mcp`. Interactive tools connect to the same account over MCP with OAuth.

## Cost to try

Vilix AI has a free plan that is free forever, plus a 7-day free trial of Pro with no credit card. You can export all of your memory at any time, or delete individual memories or the whole account.

## FAQ

**Does WhatsApp itself store memory for my AI agent?**
No. WhatsApp delivers messages and keeps the chat visible. Your agent receives one message per webhook, and nothing in that delivery loads earlier conversations into your model call.

**What is the simplest key to store WhatsApp memory against?**
The customer's phone number. It arrives in every webhook payload and stays stable across conversations.

**Can one memory serve WhatsApp and my other AI tools?**
Yes, if the memory lives outside any single tool. A shared memory layer connected over MCP can be read and written by your WhatsApp agent and by the other AI tools on your account.

## Links

- Episode: Agentic Pod EP33 (YouTube link added when published)
- Full post: https://vilix.ai/blog/how-to-give-whatsapp-ai-agent-memory-between-conversations
- Vilix AI: https://vilix.ai
- Get started: https://vilix.ai/get-started
