# AI:GO 2.0: Run Local AI Agents on Your Own PC, No Cloud Required

AI:GO 2.0, from Lablup in Seoul, launched on November 27, 2026. It runs local language, vision, image and speech AI models directly on your PC in a single app, with no reliance on external cloud services, and it works as an agent workspace: real files, tools and schedules, while sensitive files stay on your machine. It was first unveiled as Backend.AI:GO at CES 2026 in January, and this version settles on the name AI:GO.

- Free download from the official site, usable right away with no sign-up or login required.
- Runs on macOS 15 or later (Apple Silicon), Windows 10/11 (64-bit), and Linux.
- One chat window for text, image generation, speech transcription and real-time document translation.
- Watch the episode: https://youtu.be/A-arcDHXC1Y

## Download and first run

1. Get the desktop app from the official site: https://go.backend.ai
2. Install it on macOS 15+ (Apple Silicon), Windows 10/11 (64-bit), or Linux.
3. Open it. There is no sign-up or login step for the free app.
4. Run the built-in benchmark. It measures how fast each model runs on your own PC, so you pick a model based on your hardware, not a guess.
5. Let the settings tune themselves per use case: chat, long documents, or agents each get their own tuning.

## How the workspace works

- Chat-centric home: one chat window handles text conversations, image generation, speech transcription, and real-time document translation.
- Local models only: language, vision, image and speech models run on your PC. Nothing needs an external cloud service.
- If you already juggle local runners such as Ollama, LM Studio, llama.cpp or vLLM, AI:GO is the one-app alternative: the whole local stack, agents included, in a single place.

## Cowork: the agent that edits your files

1. Attach a working folder in the chat window. Cowork starts as soon as the folder is attached.
2. Cowork reads and edits files directly, only within that permitted folder.
3. Good first jobs, straight from the launch notes: organizing large numbers of files, restructuring code, and analyzing data.
4. For complex work, use Squad: role-specific agents grouped into a team that tackles the task together.

## Approvals and the sandbox

- One consistent tool approval policy applies across every execution path: chat, agents, automations, and external tool connections.
- Sensitive actions such as payments, orders, sending, and deletion require your approval every time.
- Python code and shell commands run in a built-in OS sandbox based on Microsoft Execution Containers (MXC).
- Agent code can read and write files only within a designated sandbox folder. Access to any other folder is blocked by default, and so are network connections.
- MXC uses each operating system's own process isolation under one security policy, so it works without separate Docker containers or virtual machines on macOS, Linux, and recent versions of Windows.

## A real workflow

1. Morning: one folder holds 400 messy research files, notes, PDFs and screenshots.
2. Attach that folder to Cowork and ask for sorted folders, renamed files, and a summary sheet. It edits the real files, in that folder.
3. Split the follow-up with Squad: one agent drafts, another checks the numbers.
4. The contracts folder next door stays untouched: never attached, never opened, and the sandbox blocks wandering code anyway.

## The enterprise route

- AI:GO Enterprise is the commercial edition: organization-wide policy deployment, device registration, and air-gapped operation.
- When a job needs a very large model, users can connect to inference sessions on Backend.AI Enterprise, Lablup's platform for operating large-scale GPU clusters, and keep working from the same screen.

## Gotchas and limits

- Local models are only as strong as your own hardware. An older laptop will feel it, and the built-in benchmark is there to tell you the truth before you commit.
- Cowork only touches the folder you attach. That is the safety model, so plan your folder layout before a big job.
- The sandbox blocks other folders and the network by default. Jobs that truly need network access or files elsewhere need a different setup.
- Big models and heavy agent teams still want big machines. Local first does not repeal physics.

## Who it is for

- Anyone whose files are too sensitive for a cloud upload, or whose cloud bill keeps growing.
- Builders who want agents that edit real files on their own machine, with approvals and a sandbox.
- Teams that need one policy across workstations, with an air-gapped option in the enterprise edition.

## Links

- Episode: https://youtu.be/A-arcDHXC1Y
- Lablup launch announcement, November 27, 2026: https://www.lablup.com/press/2026-11-lablup-launches-ai-go-v2
- AI:GO official site and download: https://go.backend.ai
