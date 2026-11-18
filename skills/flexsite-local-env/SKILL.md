---
name: flexsite-local-env
description: Set up and run a FlexSite environment locally with the repository's Lando or DDEV config and the FlexSite CLI. Use when the user wants to run their FlexSite site on their machine or pull the database and files.
---

# Run a FlexSite environment locally

1. Call `get_local_setup` with the `environmentId` (from `get_project_status`). It returns the CMS, the clone URL, the local development tools chosen for the project, whether database backups exist to sync from, the commands to run in order, and a `note`.
2. Run the commands `get_local_setup` returned, in order, exactly as returned. Do not invent extra FlexSite CLI commands or flags.
3. Sign in the FlexSite CLI on the host machine before the first returned command that needs it. The token needs `repo:write` to clone. When the commands include `flexsite sync`, it also needs `backup:read`, `files:read`, `backup:create`, and `files:create`, because sync creates the first backup when none exists.
   1. Run `flexsite auth status` on the host machine, not inside Lando or DDEV.
      - `flexsite` is not found: tell the user to install it with `npm install -g @flexsiteio/cli` (Node.js 22 or later), then run it again.
      - Signed in to the organization that owns the project: continue. Do not ask the user to log in again.
      - Signed in to a different organization, and the status lists a token for the project's organization: tell the user to run `flexsite auth use` and pick that organization.
      - Not signed in, or no saved token for the project's organization: continue with 2.
   2. If the user has no FlexSite account, send them to https://flexsite.io/signin?tab=signup and stop until they say the account exists.
   3. Ask once: "Will you use the FlexSite CLI for one organization, or more than one?" Recommend one login for one organization; it covers every project in that organization. For several organizations, one token per organization, then `flexsite auth use` to switch. Do not suggest a token per project.
   4. Send them to https://flexsite.io/dashboard/profile/tokens to create a token with the scopes above, then tell them to run `flexsite auth login` in their own terminal. Do not ask them to paste the token into chat. Do not run `flexsite auth login` yourself if it would prompt for the token in this session.
   5. When they say they have logged in, run `flexsite auth status` once. Continue only if the default organization owns the project.

   `lando flexsite auth login` and `ddev exec flexsite auth login` sign in the container so `lando flexsite sync` or `ddev exec flexsite sync` can run. They do not sign in the host machine's CLI that git uses. The first time the user syncs from inside Lando or DDEV, tell them the container has its own login, even when the host machine is already signed in.
4. If the `note` says the project lists more than one local tool, ask the user which one to use (`lando start` or `ddev start`) and run it in the cloned repository. Do not write a new config.
5. Run `flexsite sync` when it is in the returned commands, after DDEV or Lando is started. If there is no backup, the command says so and asks to create one. Agree, so the local site can be filled from that backup. Do not skip sync because no backup exists yet.
6. When the CMS is Drupal, run `drush status` through the local tool (`ddev drush status` or `lando drush status`) and report the Drupal version, database connection, and bootstrap result.
7. Tell the user what they can ask next. Show the list from "What you can ask me now" in the flexsite-site-content skill, then these day-2 commands:
   - Deploy: commit and push the branch mapped to the environment (the flexsite-deploy skill). Pushing is the deploy.
   - Refresh local data from the cloud: `flexsite sync`, or `flexsite db pull` / `flexsite files pull` for one of them.
   - Run drush on a cloud environment: `flexsite drush <project>.<env> -- <command>`, for example `flexsite drush mysite.dev -- cr`.

Do not commit local database dumps, files backups, or credentials.

Hosting a Drupal site that is not on FlexSite yet is the flexsite-migrate skill, not this one.
