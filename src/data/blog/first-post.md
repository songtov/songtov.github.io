---
author: songtov
pubDatetime: 2026-04-27T00:00:00Z
title: 첫 글 — Claude로 블로그 글 쓰는 법
slug: first-post
featured: true
draft: false
tags:
  - meta
description: AstroPaper 블로그를 셋업하고, Claude로 새 글을 일관된 형식으로 생성하는 방법을 정리합니다.
---

이 블로그는 [AstroPaper](https://github.com/satnaing/astro-paper) 테마와 GitHub Pages로 운영됩니다. 새 글은 `src/data/blog/` 아래에 마크다운 파일을 추가하면 됩니다.

## Claude로 새 글 생성할 때 쓰는 프롬프트 형식

> 다음 frontmatter 규격에 맞춰 `src/data/blog/<slug>.md` 파일을 만들어줘. 본문은 마크다운으로, 코드 블록과 소제목을 자유롭게 사용해.

필수 frontmatter:

```yaml
---
author: songtov
pubDatetime: 2026-04-27T00:00:00Z   # ISO 8601, UTC
title: 글 제목
slug: url-slug                       # 파일명과 일치시키는 게 무난
featured: false                      # 메인 상단 노출 여부
draft: false
tags:
  - tag1
  - tag2
description: 한두 문장 요약 (OG/검색용)
---
```

선택 필드: `modDatetime`, `ogImage`, `canonicalURL`.

## 로컬 미리보기

```bash
pnpm install
pnpm dev          # http://localhost:4321
pnpm build        # 정적 빌드 → dist/
```

`main` 브랜치로 push하면 GitHub Actions가 빌드해서 https://songtov.github.io 에 배포합니다.
