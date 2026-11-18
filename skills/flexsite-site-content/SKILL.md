---
name: flexsite-site-content
description: Manage content on a FlexSite Drupal site with AI agent access - add drafts, edit and publish content, add menu links and DAM images, build Canvas pages, clear the cache, or make the same change on many sites. Use when the user asks to add or change a page, post, article, event, or other content, or asks what they can do with their FlexSite site.
---

# Manage content on a FlexSite Drupal site

Work on a non-production environment unless the user named production. Say which environment you changed.

## What you can ask me now

Use this when a site is ready, or when the user asks what they can do with it.

1. Call `list_site_capabilities` with the `projectId`. For each environment it reports whether AI agent access is on, whether it is production, the user's role, and the operations that role allows.
2. If AI agent access is off for the environment, do not list site tools. Tell the user how to turn it on: for a new project, `aiMcp: true` when it is created (the flexsite-new-project skill); for an existing site, the environment page in the FlexSite dashboard, **AI agent access**, then **Finish setup**, after the AI packages are deployed. It needs Drupal core 11.4 or later.
3. Otherwise call `describe_site` with the `environmentId`, then show the list below. Keep only the lines the user's role allows on that environment. Show the Canvas line only when `describe_site` says the site has Canvas.

Show it to the user like this, one line each, with a sentence they could say:

- Create a draft page, post or event: "Add a draft blog post about our spring opening."
- Edit existing content: "Change the opening hours on the Contact page."
- Publish when you are ready: "Publish the spring opening post."
- Add a menu link: "Add About us to the main menu."
- Add an image from FlexSite DAM: "Add the team photo from the DAM to the media library."
- Build a designed page (Canvas): "Build a landing page for the spring sale with a hero and three cards."
- Clear the cache: "Clear the cache on dev."
- Make the same change on many sites: "Add a Privacy page to every dev site." I show you a dry run first.
- Check health, deploys and backups: "Is my site healthy?", "Did the last deploy finish?", "When was the last database backup?"

Add: health, deploy and backup questions also work from ChatGPT or Claude. Add the FlexSite connector `https://mcp.flexsite.io/mcp` there and sign in on the FlexSite page that opens.

For the health line use `get_project_status`, `list_project_issues`, `get_latest_deployment`, `get_deployment_status`, `list_database_backups` and `get_project_quality_metrics`.

## 1. Pick the content type

1. Call `describe_site` with the `environmentId`. It lists the content types with labels, descriptions and whether you may create them, the text formats, and whether the site has Canvas.
2. Match the user's words ("post", "page", "article") to a content type's label and description. If two fit, or none does, ask the user.

## 2. Fill its fields

3. Call `get_content_type_fields` with the `environmentId` and `contentType`. Use its field names and item formats. Ask the user for required fields you cannot fill from their request.

## 3. Create a draft

4. Call `create_content` with the `contentType`, `title`, and `fields` (or `body`). Leave `published` unset so it is saved as an **unpublished draft**.
5. Report the new item's id and URL, and any field the result says came back empty.

## Editing content

Find the item with `list_content` (filter by `contentType` when you know it), and read it with `get_content`. If more than one item could be the one the user means, ask. Call `update_content` with only the title, body or fields that change; `body` replaces the whole body. It saves a new revision, so the old values stay in the revision history. Canvas pages accept a title only.

## Designed pages (Canvas only)

Use `build_canvas_page` **only when the user asked for a designed page** (a landing page, or a page built from named components), and `describe_site` says the site has Canvas. First call `list_canvas_components`, then call it again with `componentIds` to get their props and slots. Build the page; the layout stays a draft to review in the Canvas editor. If the build stops early, continue with the `pageId` it returned.

## Menu links

Call `add_menu_link` with the link text and a path on the same site, such as `/about`. It goes in the main menu unless the user named another menu.

## Images from FlexSite DAM

Call `add_media_from_dam` with the DAM `assetId`. It adds the image to the site's media library; it does not place it on a page. Ask the user for the asset id if they did not give it.

## Cache

Call `clear_site_cache` when the user asks, or when a change does not show on the site. It does not change content.

## Many sites at once

1. Call `run_on_sites` with `dryRun: true`. Show the user the plan: the sites that will run, the ones skipped and why, and any production sites.
2. Only after the user agrees, call it again with `dryRun: false`, the same operation and arguments, and the `confirmationId`. Pass `confirmProduction: true` only when the plan has production sites and the user confirmed them.
3. Report progress and per-site results with `get_site_operation` when the user asks. Do not poll.

## Publishing

**Do not publish unless the user asked to publish.** Use `publish_content`. Publishing needs the admin role. If the user's role does not allow it, say so.
