# FlexSite for Cursor and Claude Code

Host your Drupal or WordPress site on [FlexSite](https://flexsite.io) without leaving your editor. Ask your agent to create a site, run it locally in DDEV or Lando, deploy it with a git push, or move an existing Drupal site from Acquia, Pantheon or anywhere else. The agent walks you through each step, shows you what it will do, and waits for your yes before it spends credits, pushes code, or replaces a database.

## What you can ask

| Say this | What happens |
|---|---|
| "Create a new Drupal 11 site on FlexSite" | Lists project types and templates, shows a preview with the credits it costs, creates the project after you confirm, then clones its repository. |
| "Run my FlexSite site locally" | Starts the project's DDEV or Lando config, pulls the database and files, and checks `drush status`. |
| "Deploy my changes to dev" | Checks that no rollout is running, pushes the branch mapped to dev, and reports the pipeline, build, image and rollout. If the pipeline fails, it reads the job log and proposes a fix. |
| "Move this Drupal site to FlexSite" | Checks `composer.json` for hosting blockers, creates the project, merges the platform files, deploys the code, then imports the database and files after you confirm each one. |
| "Add a draft blog post about our spring opening" | Creates unpublished content on a Drupal site that has AI agent access turned on. It can also edit content, publish it, add menu links and DAM images, build Canvas pages, and clear the cache. |
| "Is my site healthy? Did the last deploy finish?" | Reads environment health, issues, deployments and backups. |

WordPress: creating a WordPress project, running it locally and deploying it work today. Moving an existing site and editing content are Drupal only for now.

## Install

### Claude Code

```
/plugin marketplace add Flexsite-io/flexsite-plugin
/plugin install flexsite@flexsite
```

### Cursor

Open **Customize** in the sidebar, search for **FlexSite**, and select **Install**.

To try a local copy, put this folder at `~/.cursor/plugins/local/flexsite` and reload the window.

## Before you start

- A FlexSite account. [Sign up](https://flexsite.io/signin?tab=signup).
- git, and Node.js 22 or later for the FlexSite CLI: `npm install -g @flexsiteio/cli`. The agent tells you when it needs it.
- DDEV or Lando, only to run the site locally.

The first time the agent uses a FlexSite tool, your editor opens a FlexSite sign-in page. You approve the connection there. No token is pasted into chat.

For git, the database and files, the CLI uses a personal access token that you create at [flexsite.io/dashboard/profile/tokens](https://flexsite.io/dashboard/profile/tokens) and enter yourself with `flexsite auth login`. The agent names the scopes each task needs:

| Task | Token scopes |
|---|---|
| Clone and deploy | `repo:write` |
| Pull the database and files to your machine | `repo:write`, `backup:read`, `files:read`, `backup:create`, `files:create` |
| Move an existing site | `repo:write`, `database:import`, `files:import` |

## What the plugin contains, runs and sends

The plugin is markdown and JSON only: five skills, one Cursor rule, and the MCP server configuration. It has no hooks, scripts or binaries, and it collects no telemetry.

- **MCP server.** It connects to `https://agents.flexsite.io/mcp`, the FlexSite server, over HTTPS with OAuth. Tool calls send FlexSite what the task needs: project and environment ids, the choices for a new project, the `composer.json` of a site you are moving, and the content you ask it to create or change.
- **Commands on your machine.** The skills ask your agent to run `flexsite` (the FlexSite CLI), `git`, `ddev` or `lando`, `composer` and `drush` in your project. Your editor's own approval rules apply to each command.
- **Code, database and files.** Code goes to your project's FlexSite repository when you push. A database or files archive is uploaded only after you confirm the import, because it replaces the environment's data.

You can revoke the editor's connection at any time under **Profile → Connected apps** in the [FlexSite dashboard](https://flexsite.io/dashboard). Read the [privacy policy](https://flexsite.io/privacy-policy).

## Skills

| Skill | Use |
|---|---|
| `flexsite-new-project` | Create a Drupal or WordPress project and clone its repository |
| `flexsite-local-env` | Run a FlexSite environment locally with DDEV or Lando |
| `flexsite-deploy` | Deploy by pushing the mapped branch, check the rollout, fix a failed pipeline |
| `flexsite-migrate` | Move an existing Drupal site onto FlexSite |
| `flexsite-site-content` | Manage content on a Drupal site with AI agent access |

## Support

Write to [contact@flexsite.io](mailto:contact@flexsite.io), or open an issue in this repository.

## License

[MIT](LICENSE)
