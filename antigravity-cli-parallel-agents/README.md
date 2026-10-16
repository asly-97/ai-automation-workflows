# How do you run coding agents in parallel with Google Antigravity?

A practical first-day guide to Google Antigravity 2.0: what it is, how to install the CLI, how to start two agents at once, and the two gotchas (permission modes and quota). Companion to the Agentic Pod episode "How Do You Run Coding Agents in Parallel With Google Antigravity?".

Facts in this guide come from the official download page (https://antigravity.google/download) and the Google Antigravity Wikipedia article (https://en.wikipedia.org/wiki/Google_Antigravity), checked on 2026-10-15. Check Google's own docs before relying on a detail, because versions and limits change.

## What Antigravity 2.0 is

- A standalone desktop app for running and managing several coding agents, shown at Google I/O on 19 May 2026. It works without an IDE.
- The original editor continues as a separate Antigravity IDE.
- An Antigravity CLI that replaced the Gemini CLI for consumers.
- An Antigravity SDK (Python) for building agents you host yourself.
- Versions on the download page on 2026-10-15: app 2.23.1, CLI 1.3.6.

## Install

CLI, macOS, Linux and Googlebook (Linux terminal), as listed on the download page:

```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
```

Windows PowerShell:

```powershell
irm https://antigravity.google/cli/install.ps1 | iex
```

Read the script before you pipe it into a shell if you prefer: open the URL in a browser first.

Desktop app requirements (download page): macOS 12 or later (Apple Silicon or Intel), Windows 10 64-bit or later (x64 or ARM64), Linux with glibc 2.28 or newer (x64 or ARM64), or Googlebook via Google Play.

## First day checklist

1. Open the tool inside a git repository you can afford to experiment in. Create a branch first.
2. Start in **Strict** mode. Strict asks for approval and runs terminal commands in a sandbox with restricted network access. **Turbo** runs terminal commands without asking.
3. Give agent 1 a small, checkable job, for example: "add tests for the date parsing module".
4. Start agent 2 on a different task that touches different files, for example a README fix. Different files means no merge fights.
5. Read the Artifacts each agent produces (task lists, implementation plans, walkthroughs, screenshots, browser recordings) before you accept any change.
6. Review the diff of each agent separately. Merge one at a time.

## Quota

- Free users get a usage quota that refreshes weekly.
- Google AI Pro and Ultra subscribers get larger quotas that refresh every five hours, and can buy extra credits.
- Every agent you start spends from your quota, so start with one or two small tasks and see how far a week of free usage goes.

## Who it is for

- Good fit: anyone who wants to learn parallel agents for free, and likes seeing what each agent did through Artifacts.
- Check first: enterprise accounts can use the app and the CLI with Gemini Enterprise, but the IDE is not supported for enterprise customers.
- If you are happy with your current setup: try one small task in Strict mode and compare before you change your workflow.

## Sources

- Antigravity downloads: https://antigravity.google/download
- Google Antigravity (Wikipedia): https://en.wikipedia.org/wiki/Google_Antigravity
