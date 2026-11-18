---
name: flexsite-deploy
description: Deploy code to a FlexSite environment by pushing its mapped FlexLab branch, then check the rollout and fix a failed pipeline. Use when the user asks to deploy, ship, or push changes to a FlexSite site.
---

# Deploy to a FlexSite environment

On FlexSite, **pushing the branch mapped to an environment is the deploy.** Push only when the user asked to deploy.

## 1. Find the branch

1. Call `get_repository` with the `projectId`. It lists the branch of each environment. Confirm the target environment with the user. Production deserves an explicit yes.
2. Check that the local repository is on that branch and that its remote is the FlexLab clone URL. If git asks for a credential, sign in the FlexSite CLI on the host machine (below), then tell the user to run `flexsite git setup`. Never ask for a token.

### Sign in (only when git asks for a credential)

The token needs `repo:write` only.

1. Run `flexsite auth status` on the host machine, not inside Lando or DDEV.
   - `flexsite` is not found: tell the user to install it with `npm install -g @flexsiteio/cli` (Node.js 22 or later), then run it again.
   - Signed in to the organization that owns the project: do not ask the user to log in again. Go to `flexsite git setup`.
   - Signed in to a different organization, and the status lists a token for the project's organization: tell the user to run `flexsite auth use` and pick that organization.
   - Not signed in, or no saved token for the project's organization: continue with 2.
2. If the user has no FlexSite account, send them to https://flexsite.io/signin?tab=signup and stop until they say the account exists.
3. Ask once: "Will you use the FlexSite CLI for one organization, or more than one?" Recommend one login for one organization; it covers every project in that organization. For several organizations, one token per organization, then `flexsite auth use` to switch. Do not suggest a token per project.
4. Send them to https://flexsite.io/dashboard/profile/tokens to create a token with `repo:write`, then tell them to run `flexsite auth login` in their own terminal. Do not ask them to paste the token into chat. Do not run `flexsite auth login` yourself if it would prompt for the token in this session.
5. When they say they have logged in, run `flexsite auth status` once. Continue only if the default organization owns the project. Then tell them to run `flexsite git setup`.

`lando flexsite auth login` and `ddev exec flexsite auth login` sign in the container only. Git on the host machine uses the host machine's login.

## 2. Make sure nothing is already rolling out

3. Call `get_deployment_status` with the environment's `environmentId`. If any stage is `running` or `pending`, tell the user and **do not push**. A second push while a rollout is running is not allowed. Ask whether they want to wait, and check again only when they say so.

## 3. Commit and push

4. Commit the changes on the mapped branch. Do not commit secrets, `.env` files, `settings.php` credentials, or tokens.
5. Push the branch. Note the commit SHA you pushed (`git rev-parse HEAD`).

## 4. Check the rollout

6. Call `get_deployment_status` with the `environmentId` and that `commitSha`. Report each stage: pipeline, build, image, rollout, post-deploy.
   - All `done`: the deploy succeeded. Give the environment URL.
   - Some stage still `running` or `pending`: tell the user. Call again only if the user asks or wants to wait. Do not poll.
   - A stage `failed`: go to step 7.

## 5. Fix a failed pipeline

7. Read the failed job's log included in the `get_deployment_status` result (the last lines of the failed FlexLab job). Find the cause in the code (composer errors, failing tests, build steps) and fix it in the repository.
8. Show the user the fix. When they agree, commit and push again (back to step 3: confirm nothing is still rolling out first), then check the new commit SHA.

`get_deployment_status` reads the state once. It does not wait and does not start a deployment.
