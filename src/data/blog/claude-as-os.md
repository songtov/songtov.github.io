---
author: Chiho Song
pubDatetime: 2026-05-02T00:00:00Z
title: Claude를 OS처럼 사용하는 방법
slug: claude-as-os
featured: true
draft: false
tags:
  - claude
  - cowork
  - workflow
  - obsidian
  - ai-tools
description: Karpathy의 LLM Knowledge Base 방법론과 Claude Cowork로 만드는 개인 AI OS
---

Andrej Karpathy가 최근 자신의 워크플로우를 공개했다. 요약하면 이렇다.

> raw 데이터를 수집하면, LLM이 그걸 `.md` wiki로 "컴파일"한다. 사람은 wiki를 직접 건드리지 않는다. LLM의 영역이다.

나는 이 구조에 하나를 더 얹었다. Claude를 OS처럼 쓰는 것.

---

## Karpathy가 설명한 구조

1. **Ingest** — 아티클, 논문, 레포, 이미지를 `raw/` 디렉토리에 모은다.
2. **Compile** — LLM이 `raw/`를 읽고 `.md` wiki로 정리한다. 요약, 백링크, 개념별 아티클.
3. **IDE** — Obsidian에서 wiki를 열어 탐색한다. LLM이 쓴 내용을 사람이 보는 뷰어.
4. **Q&A** — wiki가 충분히 커지면 LLM 에이전트에게 복잡한 질문을 던진다. LLM이 인덱스를 자동 유지하기 때문에 RAG 없이도 된다.
5. **Output** — 답변을 markdown, 슬라이드, 이미지로 렌더링해 Obsidian에서 본다.
6. **Linting** — 주기적으로 wiki 전체를 헬스체크한다. 불일치 데이터, 빠진 정보, 새 아티클 후보 발굴.

루프의 핵심: **쿼리가 wiki를 강화한다.** 내가 물어볼 때마다 그 결과가 다시 wiki에 쌓인다.

---

## LLM OS 관점

이 구조가 OS처럼 보이는 이유가 있다.

- **LLM** = 커널. 작업을 조율하고, 툴을 호출하고, 데이터를 변환한다.
- **폴더** = 파일시스템. wiki와 raw 데이터가 persistent storage다.
- **Obsidian** = 디스플레이. 파일시스템의 내용을 사람이 읽는 뷰어.

기존 OS가 프로그램을 실행하듯, 이 구조는 **의도를 실행한다.**

---

## Claude Cowork로 구현하는 법

[Claude Cowork](https://claude.com/product/cowork)는 Claude Desktop에 탑재된 에이전트 실행 환경이다. 터미널 없이 Claude Code와 같은 agentic 능력을 쓸 수 있다.

핵심 개념은 **폴더를 VM처럼 쓰는 것**이다.

1. **`Claude OS/` 폴더 생성** — 이 폴더가 격리된 작업 공간이 된다. 여기에 `raw/`, wiki, 이메일 초안, 리포트 등 모든 산출물이 쌓인다.
2. **Cowork에서 폴더 지정** — Cowork 프로젝트의 루트로 `Claude OS/`를 연결한다. Claude는 이 폴더 안에서만 읽고 쓴다.
3. **규칙과 워크플로우 정의** — 프로젝트 instructions에 규칙을 적는다. 파일 네이밍 컨벤션, 어떤 작업을 어떤 형식으로 처리할지, 반복 태스크 스케줄 등.
4. **에이전트 실행** — 원하는 결과만 말하면 된다. Claude가 계획을 세우고, 병렬 워크스트림을 돌리고, 파일을 직접 생성한다. Obsidian으로 결과를 열면 된다.

이렇게 되면 `Claude OS/` 폴더는 단순한 디렉토리가 아니다. 규칙이 명세되고, 에이전트가 상주하며, 산출물이 누적되는 **VM에 가까운 개인 컴퓨팅 환경**이 된다.

