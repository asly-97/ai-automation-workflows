# Steering Agents From Your Phone (Agent Inbox)

From ChatGPT or Claude on your phone, connected to Vilix AI:

```
Message my agents through Vilix AI: pause competitor price alerts until Monday,
the supplier is updating their catalog. Urgent, only for the OpenClaw heartbeat.
```

The assistant calls:

```
message_agents(
  body="Pause competitor price alerts until Monday. Supplier is updating the catalog.",
  urgency="urgent",
  scope_type="source",
  scope_value="OpenClaw heartbeat",
  ttl_hours=96
)
```

On its next run the heartbeat's `get_context` includes the message in its `inbox`. It follows it and calls `ack_message`. No prompt edits, no SSH into the server.

Other good inbox uses:
- "Codex nightly: the payments folder is frozen this week, don't touch it." (`scope_type: source`)
- "Everyone: we switched the CRM from HubSpot to Attio today." (`scope_type: global`)
- "Northstar project agents: launch moved to Friday." (`scope_type: project`)
