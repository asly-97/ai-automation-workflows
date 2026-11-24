# Claude Ship: AI agents that review, fix and approve your pull requests

Claude Ship is Anthropic's suite of three agents for pull requests on GitHub, announced on November 23, 2026. Code Review is generally available and flags bugs and issues in proposed changes. PR Steward, in beta, fixes failing CI checks, resolves merge conflicts and answers reviewer feedback. Claude Approvals, also in beta, reads a policy file your team stores in the repo and approves routine changes on its own, sending riskier updates to a human. It is rolling out to Claude Team and Enterprise plans for GitHub repositories.

## The three agents

- **Code Review** (generally available): scans a pull request through multiple lenses, adversarially verifies its findings, categorises them, and posts them ranked by severity.
- **PR Steward** (beta): works to keep a pull request green. It steps in on failing CI checks, merge conflicts, and feedback from Code Review, other tools, or colleagues. After each push it comments on what it changed and how to undo it. It cannot approve or merge pull requests, so fixing and deciding stay separate.
- **Claude Approvals** (beta): applies an approval policy the team writes as a file in the repository (`.claude/approvals.yml` on the default branch). It approves routine changes the policy allows and reports who still needs to approve anything riskier.

Claude Ship works with your existing branch protection and required checks rather than replacing them. Teams can later require the green light from Claude Approvals through branch protection.

## Way 1: /ship on your own pull request

For individual developers, no admin setup is needed:

1. Finish a change locally in Claude Code.
2. Run `/ship`.
3. Claude pushes the branch, opens the GitHub pull request, runs a review, and keeps working on feedback and failing checks within your permissions and the repo's branch rules.
4. Read the review findings and the steward's fix comments. Every fix comment explains what changed and how to undo it.
5. Merge when the pull request is green and you are happy with the diff.

## Way 2: enable it across a repository

For teams on Claude Team and Enterprise plans:

1. Start with one repository, not the whole organisation.
2. An admin turns on Code Review and PR Steward for that repository (PR Steward in manual mode first is the cautious start).
3. Add the approvals policy file (see below) to the default branch. Claude Approvals starts when a connected repository has a policy file on its default branch, and an admin can turn it off.
4. Run Claude Approvals in observe-only mode first: the agent reports what it would approve, humans still decide, and you compare the two until the policy matches your real standards.
5. Only then consider requiring the Claude Approvals green light in branch protection.

## Writing an approval policy file

The policy lives in the repo at `.claude/approvals.yml` on the default branch, written in plain language rules. Keep it concrete:

- Name the routine changes that may be approved on their own: for example docs-only changes, test-only changes, and small dependency bumps.
- Name the changes that always need a human: anything touching authentication, payments, migrations, or public APIs.
- Name who the human is for each risky area, so the agent can report exactly who still needs to approve.
- Keep the file under review like code. A loose policy is the whole risk surface of auto-approval.

## A real workflow

1. A developer pushes a small fix and runs `/ship`.
2. Code Review posts findings within minutes, ranked by severity.
3. PR Steward fixes the real bug and the failing CI check, commenting after each push with the change and the undo path.
4. Claude Approvals checks the change against the policy: small fix, tests green, no sensitive paths touched, so it approves.
5. The developer reads the final diff and merges.

## Gotchas and limits

- Two of the three agents are in beta, so behavior may change.
- Anthropic published no figures on how often auto-approval gets things wrong, and warns that Claude Ship uses AI and may miss bugs, security vulnerabilities, or other issues. Review is a layer, not a replacement for your own checks.
- GitHub repositories only. No GitLab or Bitbucket support.
- Claude Team and Enterprise plans only, and organisations on zero data retention or HIPAA configurations cannot use Claude Ship currently.
- Usage is token based, and even observe-only approval policies can incur costs when they invoke classifiers or Code Review. Set a monthly spend limit when you roll it out.
- Anthropic's own results are promising but internal: on the Claude Code team, merged pull requests more than doubled from June to September 2026 while the share reverted fell by more than 40 percent.

## Credits

Eligible Enterprise customers can claim promotional credits of $10,000 to $50,000 depending on active Claude Code seats. Claims close January 15, 2027, and credits are valid for 30 days after activation.

## Who it is for

Teams whose AI coding tools now produce more pull requests than reviewers can comfortably clear, and who are willing to write a careful policy and start in observe-only mode. If your team has no written approval standards at all, write those first: the policy file is only as good as the standards it encodes.

## FAQ

**Can PR Steward merge my pull request?**
No. It fixes and comments, but it cannot approve or merge. Approval belongs to Claude Approvals under your policy, or to a human.

**What happens if there is no policy file?**
Claude Approvals does not start. It only runs on a connected repository with a policy file on the default branch.

**Can I turn it off?**
Yes. An admin can turn Claude Approvals off, and the repo-level agents are enabled per repository.

**What does it cost?**
Usage is billed on tokens. Code Review is part of the suite, and approval policies that invoke classifiers or a review incur token costs even in observe-only mode.

## Watch the episode

Agentic Pod EP64: Claude Ship: AI Agents That Review, Fix and Approve Your Pull Requests
https://youtu.be/_Rc55ocZkHU

Sources: https://techcrunch.com/2026/11/23/anthropic-launches-claude-ship-letting-ai-agents-review-fix-and-approve-code-changes/ , https://www.theregister.com/ai-and-ml/2026/11/24/claude-ship-puts-ai-at-the-helm-of-pull-request-approvals/5311883 , https://aimagazine.com/news/how-claude-ship-governs-code-releases-with-policy , https://apidog.com/blog/what-is-claude-ship/
