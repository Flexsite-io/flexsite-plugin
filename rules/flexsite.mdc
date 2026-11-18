---
description: FlexSite and FlexLab conventions for repositories, deploys, secrets and status checks
alwaysApply: true
---

# FlexSite

- Call the git layer **FlexLab**: the FlexLab repository, FlexLab merge requests, FlexLab pipelines. Do not call it GitLab when you talk to the user.
- Pushing the branch mapped to an environment **deploys** that environment. Treat every push to a mapped branch as a deploy, and push only when the user asked for it.
- Never commit, print, or paste secrets: GitLab or FlexLab tokens, FlexSite personal access tokens, database passwords, `settings.php` credentials, or `.env` values. Do not ask the user to paste a token into chat.
- For git access, tell the user to run `flexsite git setup`. It makes git ask the FlexSite CLI for a credential, so no token goes into the remote URL or the repository. Then clone with `flexsite git clone`.
- Do not poll. Call `get_project_status` or `get_deployment_status` again only when the user asks, or when the previous result said the step is still running and the user wants to wait. Each call reads the state once; report it and stop.

## Signing in

The FlexSite CLI keeps one login per user on the host machine, not per project. Before a task that needs the CLI (clone, push, sync, import), check it on the host machine, not inside Lando or DDEV:

1. Run `flexsite auth status`.
   - `flexsite` is not found: tell the user to install it with `npm install -g @flexsiteio/cli` (Node.js 22 or later), then run it again.
   - Signed in to the organization that owns the project: continue. Do not ask the user to log in again.
   - Signed in to a different organization, and the status lists a token for the project's organization: tell the user to run `flexsite auth use` and pick that organization.
   - Not signed in, or no saved token for the project's organization: go to step 2.
2. If the user has no FlexSite account, send them to https://flexsite.io/signin?tab=signup and stop until they say the account exists.
3. Ask once: "Will you use the FlexSite CLI for one organization, or more than one?"
   - One organization: recommend one login. It covers every project in that organization.
   - More than one: one token per organization, then `flexsite auth use` to switch between them.
   - Do not suggest a token per project.
4. Send them to https://flexsite.io/dashboard/profile/tokens. Name only the scopes the task needs (the skill lists them). Tell them to create the token there, then run `flexsite auth login` in their own terminal.
   - Do not ask them to paste the token into chat.
   - Do not run `flexsite auth login` yourself if it would prompt for the token in this session.
5. When they say they have logged in, run `flexsite auth status` once. Continue only if the default organization owns the project.

`lando flexsite auth login` and `ddev exec flexsite auth login` sign in the container, so `lando flexsite sync` or `ddev exec flexsite sync` can run. They do not sign in the host machine's CLI that git uses. For clone or push, the host machine's login is the one that matters. The first time the user syncs from inside Lando or DDEV, tell them the container has its own login, even when the host machine is already signed in.
