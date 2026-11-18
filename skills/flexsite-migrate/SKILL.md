---
name: flexsite-migrate
description: Host an existing local Drupal site on FlexSite. Use when the user asks to migrate, host, or deploy a Drupal site they already have onto FlexSite. For a site that is already a FlexSite project, use flexsite-deploy instead.
---

# Host an existing Drupal site on FlexSite

The site is a Drupal project on the user's machine, often from Pantheon or Acquia. It is not a FlexSite project yet. If it already is, stop and use the flexsite-deploy skill.

`<tool>` below is `ddev` or `lando`, whichever the local site uses.

## 1. Check the project

1. The working directory (or the directory the user named) must contain `composer.json`. Read it.
2. Call `check_drupal_hosting_compatibility` with that `composer.json`. Tell the user what it reports, including every item in `findings` (`web_root`, `platform_modules`, `search_backend`, `build_scripts`). Steps 15 and 17 act on them.
3. Read the Drupal core version from `composer.json` (`drupal/core-recommended` or `drupal/core`). Before anything else, ask whether they want AI agent access. It needs Drupal core 11.4 or later and Drush 13.8 or later. On an older core the site still moves to FlexSite, but the site tools stay off until core is upgraded.

## 2. Create the FlexSite project

4. Call `list_organizations` if you do not already know which organization to use, and ask the user which one. Ask for the project name.
5. Call `list_project_types`. Match the Drupal version to a project type. Ask when two fit.
6. Call `list_installable_templates` for that type. For an existing site, do not pick a marketplace site template or theme unless the user asks for one.
7. Call `preview_project_create` and show the credits and any broken rule. Stop if a rule is broken. Do not pass `aiMcp` for a migration: AI agent access is turned on after the database import (step 24).
8. Do not call `create_project` until the user has seen the preview and agreed. Creating spends credits.
9. Call `create_project` with those inputs. Then `get_project_status`.
   - `ready: true`: continue, and keep the dev environment id and URL.
   - An environment failed: quote `failureReason` and stop.
   - Not ready yet: tell the user. Call `get_project_status` again only when they ask. Do not poll.
   If the user already has a project id for this site, skip steps 4 to 9 and start from `get_project_status`.

## 3. Put the platform files in the local git history

10. Call `get_repository`. If `provisioning` is true, tell the user and stop until they ask you to check again. Note the clone URL and the branch mapped to dev. Pushing that branch deploys dev.
11. Sign in the FlexSite CLI on the host machine. The token needs `repo:write`, `database:import`, and `files:import`.
    1. Run `flexsite auth status` on the host machine, not inside Lando or DDEV.
       - `flexsite` is not found: tell the user to install it with `npm install -g @flexsiteio/cli` (Node.js 22 or later), then run it again.
       - Signed in to the organization that owns the project: continue to step 12. Do not ask the user to log in again.
       - Signed in to a different organization, and the status lists a token for the project's organization: tell the user to run `flexsite auth use` and pick that organization.
       - Not signed in, or no saved token for the project's organization: continue with 2.
    2. If the user has no FlexSite account, send them to https://flexsite.io/signin?tab=signup and stop until they say the account exists.
    3. Ask once: "Will you use the FlexSite CLI for one organization, or more than one?" Recommend one login for one organization; it covers every project in that organization. For several organizations, one token per organization, then `flexsite auth use` to switch. Do not suggest a token per project.
    4. Send them to https://flexsite.io/dashboard/profile/tokens to create a token with those three scopes, then tell them to run `flexsite auth login` in their own terminal. Do not ask them to paste the token into chat. Do not run `flexsite auth login` yourself if it would prompt for the token in this session.
    5. When they say they have logged in, run `flexsite auth status` once. Continue only if the default organization owns the project.

    `lando flexsite auth login` and `ddev exec flexsite auth login` sign in the container only. Git and the `flexsite db push` / `flexsite files push` commands below run on the host machine and use the host machine's login.
12. Run `flexsite git setup`.
13. In the local Drupal directory, add a remote named `flexlab` with the clone URL if it is not there, then `git fetch flexlab`.
14. Merge the mapped branch with `git merge flexlab/<branch> --allow-unrelated-histories`. Keep the local `composer.json`, `composer.lock`, web root, modules, themes, and config. When a platform file conflicts, take the FlexLab side: `.gitlab-ci.yml`, `Dockerfile`, `docker/`, `.flexsite.yml`, `web/health.php`, and the FlexSite include in `settings.php`. Do not commit `settings.php` secrets, `.env`, or tokens.
15. FlexLab builds from `web/`. When `findings` has `web_root` (for example `docroot`):
    - `git mv <root> web`. If the merge already created `web/`, first move its files into the local root (`health.php`; for `sites/default/settings.php`, add the FlexSite include to the local file instead), then rename.
    - In `.gitignore`, rewrite lines that start with `/<root>/` or `<root>/` to `/web/` or `web/`.
    - Set `webroot` in `.lando.yml` or `docroot` in `.ddev/config.yaml` to `web`, and change any other `<root>/` path under `.lando*`, `.ddev/`, and `drush/`. Then `lando rebuild -y` or `ddev restart`.
16. Run `flexsite composer fix -n <project-name> --backup`. It rewrites `<root>/` paths in `composer.json`, removes platform packages, and adds the FlexSite packages and repository. Then put back what the FlexLab template ships, which the dev database already depends on:
    - Every `flexsite/*` require in `git show flexlab/<branch>:composer.json` that the local `composer.json` lacks.
    - `flexsite/fs_mcp` and `flexsite/fs_ai` at `^1.0`, when core is 11.4 or later (step 3).
    - The repository `https://packages.flexsite.io/packages.json` if it is still missing (`composer config repositories.flexsite composer https://packages.flexsite.io/packages.json`).
17. Other `findings`:
    - `platform_modules`: list the modules and ask before uninstalling them in the local site with `<tool> drush pmu <modules> -y`, before the lock update removes their code. Then remove their packages from `composer.json`. If one will not uninstall, tell the user and continue without it unless they say to stop.
      Do not uninstall `acquia_dam`, `acquia_dam_integration_links`, `media_acquiadam`, or a custom module that depends on them (for example `dam_integration`) unless the user has a replacement for those media types. Uninstalling them deletes DAM media bundles, leaves entity-reference fields with no valid target, and can 500 the theme if Twig calls a DAM function such as `render_dam_image`. Acquia DAM is a SaaS product, not hosting lock-in; keep the packages and say so. `acquia_connector`, `acquia_search`, `acquia_purge`, `memcache`, `searchstax`, and `new_relic_rpm` are hosting lock-in and should still be offered for uninstall. After `acquia_purge`, check that the Purge module still has a purger; if drush then fatals in `PurgersService::getLabels()`, report it.
    - `search_backend` and `build_scripts`: tell the user what they mean for the site. Do not change the search setup.
      FlexLab's Dockerfile runs `npm ci` at the repository root when `package.json` is present. If that file is only visual-regression tooling (Puppeteer, Backstop) and the real Sass/JS build lives under the theme, `git rm` the root `package.json` and `package-lock.json` (keep `backstop/` and the theme). Leaving Puppeteer at the root has OOM-killed the Kaniko image build (`exit 137`) after the snapshot of those 99 packages.
18. Update `composer.lock` for the changed packages only, with every name added or removed in steps 16 and 17: `<tool> composer update <package names> --with-all-dependencies`. Never run a full `composer update` on the site. The FlexLab build runs `composer install` and fails when the lock does not match. Then `<tool> composer validate` and `<tool> drush status` must both pass. If either fails, show the output and stop.
19. Commit with the message `Prepare for FlexSite hosting`, including `composer.lock`. If that commit is already on the branch, skip steps 14 to 19.

## 4. Deploy the code

20. Call `get_deployment_status` for the dev environment. If any stage is `running` or `pending`, tell the user and do not push.
21. Show the user the commit and the branch. Do not push until they agree. Pushing deploys dev.
22. Push the mapped branch. Record `git rev-parse HEAD`. Call `get_deployment_status` with that `commitSha`.
    - Every stage `done`: continue.
    - A stage `running` or `pending`: tell the user. Check again only when they ask.
    - A stage `failed`: read `failedJobLog`, fix the cause, show the fix, and go back to step 20.
    - A stage `skipped`: tell the user what the note says. A skipped build did not produce an image.

## 5. Send the database and files

23. Ask before the database import. Say that it replaces the dev environment's database.
    - Sanitize only when the user asks, or when the data is production data with personal information: `<tool> drush sql:sanitize -y`, plus any tables the user names. Sanitize a copy of the database, not the one they work on, unless they say otherwise.
    - Export to a `.sql.gz` outside the repository (`ddev export-db --file=<name>.sql.gz`, or `lando db-export <name>.sql`, which writes `<name>.sql.gz`). Always use this for a database of hundreds of MB.
    - After they agree, run `flexsite db push --env <dev environment id> --file <name>.sql.gz --yes` from the Drupal directory. Without `--file` it dumps the running DDEV or Lando site itself. If the command prints that the import is still running, tell the user and run `flexsite db push --env <id> --job <jobId>` only when they ask.
    - Then ask before the files import. Say that it replaces the dev environment's files. After they agree, run `flexsite files push --env <dev environment id> --yes`, or `--file <archive>` for an existing archive. A running import works the same way, with `flexsite files push --env <id> --job <jobId>`.
24. Run `flexsite drush <project>.dev -- cr`. Then `get_project_status` and give the user the dev URL.
    When core is 11.4 or later and the user wants AI agent access: send them to the dev environment page in the FlexSite dashboard, panel "AI agent access", "Finish setup". It takes a database backup, applies the fs_ai recipe, and connects the site. When they say it is done, call `list_site_capabilities` and continue with "What you can ask me now" in the flexsite-site-content skill.

Stop at the first failure and say which step failed. Do not commit database dumps, files archives, or credentials.
