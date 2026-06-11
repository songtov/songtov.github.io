# AGENTS.md

## Project

This repository is Chiho Song's personal Astro blog.

- Production branch: `main`
- Blog content: `src/data/blog/`
- Framework: Astro with the AstroPaper theme
- Primary language for posts: Korean
- Site focus: AI agents, LLM workflows, ontology, Python, and practical engineering notes

## Writing Guidelines

- Keep posts personal, practical, and engineering-focused.
- Prefer concrete observations from building, studying, or using tools over generic news summaries.
- Use `draft: true` while drafting.
- Set `draft: false` only when the user asks to publish.
- Prefer one post per PR unless the user explicitly asks to batch related posts.
- Use lowercase, hyphenated slugs.
- Keep frontmatter compatible with `src/content.config.ts`.
- For recent or fast-moving AI topics, verify current facts and cite sources where useful.

## Branch And PR Flow

- Treat `main` as production.
- Use short-lived branches:
  - `post/*` for blog posts
  - `feature/*` for site features
  - `fix/*` for bug fixes
  - `docs/*` for project documentation
- Open a PR for review before merging to `main` when the user asks for PR-based work.
- Do not include unrelated untracked files in commits.
- Keep commits focused and reviewable.

## Preview And Deployment

- PRs should be buildable before merge.
- When PR preview infrastructure exists, check the preview URL before publishing user-facing changes.
- Merging to `main` deploys the site through the configured GitHub Pages workflow.
- If custom-domain or preview hosting changes are made, update project guidance here.

## Verification

Run this before pushing content or site changes:

```bash
npm run build
```

For code, layout, style, or workflow changes, also run:

```bash
npm run lint
npm run format:check
```

If build verification fails because the sandbox blocks network access for Google Fonts or package metadata, retry with an escalated command and explain the reason.

## Git Hygiene

- Check `git status --short --branch` before staging.
- Stage only files related to the task.
- Do not stage `pnpm-workspace.yaml` unless the user explicitly asks for it.
- Do not rewrite history, reset, or discard user changes unless explicitly requested.
- If unrelated local changes exist, leave them alone and mention them in the final summary when relevant.

## Current Operating Preferences

- Prefer `chihosong.com` as the primary custom domain if a custom domain is configured.
- Consider Cloudflare Pages for PR previews and domain/DNS management if preview hosting is added later.
- Avoid adding Google AdSense until the blog has a stronger content base and meaningful recurring traffic.
