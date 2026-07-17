# Kevin's Lessons Learned

Kevin's Lessons Learned is a public learning notebook. Questions begin as
GitHub Issues, develop through research and practice, and become Markdown posts
reviewed in pull requests. Merging a publishable post into `main` deploys it to
GitHub Pages.

The site uses [Hugo](https://gohugo.io/) and the
[PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme. The generated
website is not stored in Git; GitHub Actions builds it from the source files.

## Repository map

- `content/posts/` contains one folder per post. Keep a post's Markdown and
  images together in the same folder.
- `content/about.md`, `content/archives.md`, and `content/search.md` define the
  permanent pages.
- `archetypes/posts.md` is the template used for new post bundles.
- `assets/css/extended/custom.css` contains the small visual adjustments made on
  top of PaperMod.
- `layouts/` contains small Hugo 0.164 compatibility overrides for the
  pinned PaperMod v8.0 release; remove them after a future theme upgrade makes
  them unnecessary.
- `.github/ISSUE_TEMPLATE/` defines learning and writing work items.
- `.github/workflows/validate.yml` previews a complete build for every pull
  request; `pages.yml` publishes only from `main`.
- `themes/PaperMod/` is a Git submodule pinned to a reviewed theme version.

## Prerequisites

Install Git and Hugo 0.164.0 or a compatible newer release. On macOS:

```sh
brew install hugo
```

Clone this repository with its theme:

```sh
git clone --recurse-submodules <repository-url>
cd <repository-directory>
```

If the repository was cloned without submodules, initialise the theme with:

```sh
git submodule update --init --recursive
```

## Preview the site

Run the development server from the repository root:

```sh
hugo server --buildDrafts
```

Hugo prints the local address to open. Draft posts are visible locally but are
excluded from production builds.

Before opening a pull request, also run the production build:

```sh
hugo build --gc --minify
```

The command creates `public/`, which is intentionally ignored by Git.

## Create a post

1. Open a **Learning item** issue to define the question and desired outcome.
2. When the idea deserves an article, open a linked **Writing item** issue.
3. Create a branch named for the issue, such as `post/42-how-something-works`.
4. Generate a page bundle:

   ```sh
   hugo new content posts/how-something-works/index.md
   ```

5. Write in `content/posts/how-something-works/index.md`. Put diagrams or
   screenshots beside it and link them with relative Markdown, for example
   `![Description](diagram.png)`.
6. Preview with drafts enabled and open a pull request that says `Closes #42`.
7. Keep `draft: true` while the post is being reviewed. Change it to
   `draft: false` only when the post is ready for the public site.
8. Merge the approved pull request into `main`; the Pages workflow publishes it
   and GitHub closes the linked issue.

Post front matter follows this shape:

```yaml
---
title: "A clear, specific title"
date: 2026-07-17T10:00:00+10:00
draft: true
description: "One or two sentences for previews and search results."
tags: ["example", "learning"]
# cover:
#   image: "cover.jpg"
#   alt: "A useful description of the cover image."
---
```

## Set up the GitHub repository

This local repository does not yet have a remote. Create an empty public GitHub
repository, then connect and push it:

```sh
git remote add origin <repository-url>
git push -u origin main
```

In the GitHub repository, open **Settings → Pages** and set **Source** to
**GitHub Actions**. The deployment workflow reads the Pages URL supplied by
GitHub, so project sites such as
`https://<username>.github.io/<repository>/` work without editing `baseURL`.
The workflow also supplies the repository URL used by each page's **Suggest
changes** link.

Create the labels referenced by the issue forms. With the GitHub CLI installed
and authenticated, run:

```sh
gh label create "type: learning" --color "1D76DB" --description "A topic or skill to learn" --force
gh label create "type: post" --color "5319E7" --description "A post to plan and write" --force
gh label create "status: backlog" --color "D4C5F9" --description "Ready to be prioritised" --force
gh label create "status: learning" --color "FBCA04" --description "Research or practice is in progress" --force
gh label create "status: drafting" --color "F9D0C4" --description "A post draft is in progress" --force
gh label create "status: review" --color "0E8A16" --description "Ready for review" --force
gh label create "status: published" --color "0052CC" --description "Published on the blog" --force
```

Issue forms can only apply labels after those labels exist. Move one status
label at a time through `backlog` → `learning` or `drafting` → `review` →
`published`.

## Update PaperMod

The theme is pinned so an upstream change cannot unexpectedly alter the site.
To review a future PaperMod release:

```sh
git -C themes/PaperMod fetch --tags
git -C themes/PaperMod checkout <new-version-tag>
git add themes/PaperMod
```

Run both local builds, inspect the site, and commit the updated submodule pointer
in its own pull request.

## Publishing rules

- Pull requests may contain drafts and always run the validation build.
- Only merges and direct pushes to `main` trigger deployment.
- Production builds exclude draft, future-dated, and expired content.
- Do not commit `public/` or generated Hugo resources.
- No analytics, comments, custom domain, or content licence is configured yet.
