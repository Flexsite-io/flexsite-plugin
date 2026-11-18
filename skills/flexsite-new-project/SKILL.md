---
name: flexsite-new-project
description: Create a new FlexSite project (Drupal or WordPress) and clone its FlexLab repository. Use when the user wants to start a new FlexSite site or project.
---

# Create a FlexSite project

Use the `flexsite` MCP server. Ask the user whenever a choice is theirs (organization, project type, template, theme, name). Do not pick one for them.

## 1. Choose what to create

1. Call `list_project_types` and show the user the project types (CMS versions) they can create.
2. When they pick one, call `list_installable_templates` with that `projectTypeId`. Show the site templates and themes, including any theme selection rules the result gives.

## 2. Preview, then create

3. For a Drupal project, ask the user whether to turn on AI agent access. It lets them add and edit the site's content by asking Cursor, Claude Code, ChatGPT or Claude. If they say yes, pass `aiMcp: true`. Do not ask for WordPress.
4. Call `preview_project_create` with the user's choices, including `aiMcp`. Show the user the result: features, credits, launch path, and any rule it reports as broken.
   - For a FlexLab (Drupal 11) project, ask the user which local tool they use and pass it as `localDevEnv` (`["ddev"]` or `["lando"]`). Leave out `requiresWorkflow`; it is for CMS Hosting only, and FlexLab is always `"no"`.
   - Leave out `url` unless the user asked for a specific one. FlexSite generates the URL when it creates the dev environment, so an empty `url` in the preview is expected.
5. **Do not call `create_project` until the user has seen the preview and confirmed it.** If the preview reports a problem, fix the inputs with the user and preview again.
6. Call `create_project` with the same inputs the user confirmed, including `aiMcp`. It creates the project. It does not run the installer.

## 3. Wait for the dev environment

7. Call `get_project_status` with the new `projectId`.
   - `ready: true`: tell the user the dev environment is ready and give its URL.
   - An environment has failed: tell the user, quoting its `failureReason`. Stop.
   - Still being created: tell the user. Call `get_project_status` again only if the user asks, or wants to wait. Do not poll in a loop.

## 4. Clone the FlexLab repository

8. Call `get_repository` with the `projectId`. If `provisioning` is true, tell the user the FlexLab repository is still being created and check again only when they ask.
9. Sign in the FlexSite CLI on the host machine before cloning. The project's organization is the one the project was created in. The token needs `repo:write`.
   1. Run `flexsite auth status` on the host machine, not inside Lando or DDEV.
      - `flexsite` is not found: tell the user to install it with `npm install -g @flexsiteio/cli` (Node.js 22 or later), then run it again.
      - Signed in to the project's organization: go to step 10. Do not ask the user to log in again.
      - Signed in to a different organization, and the status lists a token for the project's organization: tell the user to run `flexsite auth use` and pick that organization.
      - Not signed in, or no saved token for the project's organization: continue with 2.
   2. If the user has no FlexSite account, send them to https://flexsite.io/signin?tab=signup and stop until they say the account exists.
   3. Ask once: "Will you use the FlexSite CLI for one organization, or more than one?" Recommend one login for one organization; it covers every project in that organization. For several organizations, one token per organization, then `flexsite auth use` to switch. Do not suggest a token per project.
   4. Send them to https://flexsite.io/dashboard/profile/tokens to create a token with `repo:write`, then tell them to run `flexsite auth login` in their own terminal. Do not ask them to paste the token into chat. Do not run `flexsite auth login` yourself if it would prompt for the token in this session.
   5. When they say they have logged in, run `flexsite auth status` once. Continue only if the default organization owns the project.

   `lando flexsite auth login` and `ddev exec flexsite auth login` sign in the container only. Cloning uses the host machine's login.
10. Tell the user to run `flexsite git setup` (or the credential command `get_repository` returned). It makes git ask the FlexSite CLI for a credential, so no token is pasted into chat or put in the remote URL.
11. Clone with `flexsite git clone` using the clone URL from `get_repository`. Tell the user which branch maps to which environment, and that pushing a mapped branch deploys that environment.

Never ask for, print, or commit a GitLab or FlexLab token.

## 5. Tell the user what they can ask next

12. When the dev environment is ready, show the user what they can ask you now, built as described in "What you can ask me now" in the flexsite-site-content skill. Add that running the site on their machine is the flexsite-local-env skill.
