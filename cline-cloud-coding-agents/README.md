# How do you run coding agents in the cloud with Cline Cloud?

The practical guide behind the Agentic Pod episode **"How Do You Run Coding Agents in the Cloud? Cline Cloud Explained"**: what Cline Cloud is, the steps to start your first session, how the `cline/` branch and pull request flow works, the three ways to pay, a parallel workflow, and the honest limits.

## What Cline Cloud is

Cline Cloud runs Cline's coding agent in its own cloud sandbox that you control from your browser, with nothing to install. Your laptop, editor and Wi-Fi do not need to stay on. Cloud sessions started rolling out gradually in the Cline extension release v4.1.25 (October 16, 2026), where cloud tasks appear in Recent and History with a status, Cline notifies you when they finish, and they keep running after you close your editor.

## Start your first session

1. Go to https://app.cline.bot and sign in or create a free account.
2. Connect GitHub and choose which repositories Cline can access (personal repos and org repos you belong to).
3. Open the **Agents** tab, pick a repository and a branch.
4. Choose a model, then describe the task. You can paste or drag in screenshots and mockups.
5. Optional: start in **Plan** mode so Cline agrees on an approach with you before it writes code.

When the work is done, ask Cline to open a pull request, then review and merge it on GitHub.

## How the code flows

- Cline works on its own `cline/` branch, not your default branch, and commits and pushes as it goes, so progress is visible on GitHub before the task ends.
- The GitHub token for each session is scoped to the single repository you selected.
- Each session runs in its own isolated sandbox with its own copy of the repository.

## Pricing and models

- **ClineFree:** free models. The list changes over time, so check the model picker.
- **ClinePass:** a subscription for open-weight models, with 2 to 5 times the usage of standard API rates.
- **Cline usage billing:** pay as you go with Cline credits for frontier and open models.
- You can pick a model at the start and switch models mid-session. Plan and Act modes, approvals and auto-approve work as in the other Cline products.

Specific prices and credit amounts are not listed in the docs we read, so check the app before you commit.

## A workflow that uses parallel sessions

You can run up to 10 sessions at once, each with its own sandbox and branch. Four small jobs become four sessions and four pull requests:

1. "Fix the flaky login test. Make the test pass and do not touch anything else."
2. "Add dark mode to the settings page. Follow the existing theme tokens."
3. "Write docs for the export function. Docs only, no code changes."
4. "Upgrade one dependency and fix any breakage. List what changed."

Tips: say what done looks like in every prompt, use Plan mode for anything bigger than a chore, send follow-up instructions at any time to redirect a session, and check progress from the sidebar or from a mobile browser.

## Limits and gotchas

- Up to 10 sessions at once and 50 new sessions per day. If Cline's cloud is at capacity you are asked to try again later.
- If a session is idle for 15 minutes the sandbox pauses and resumes with the workspace intact when reopened.
- Rules, skills, MCP servers and connectors from your local Cline are not loaded in cloud sessions, so a setup you tuned on your laptop does not follow you.
- Cloud Agents are described as new, so expect rough edges. There is a GitHub issue about a finished cloud task still showing as "Running" (https://github.com/cline/cline/issues/14620).
- The docs do not list the software installed in the sandbox.

## Who it is for

Developers with a backlog of small, well-defined jobs, people who code on the go, and anyone who wants to try a coding agent without installing anything. For large or risky refactors, keep your own hands on the wheel and review every pull request.

## Sources

- https://cline.bot/cloud
- https://docs.cline.bot/usage/cloud-agents
- https://github.com/cline/cline/releases
