---
author: Chiho Song
pubDatetime: 2026-06-24T00:00:00Z
title: Ghostty 한국어 폰트 설정과 단축키
slug: ghostty-korean-setup
featured: false
draft: false
tags:
  - ghostty
  - terminal
  - dotfiles
  - setup
  - korean
description: Ghostty에서 한국어 폰트가 깨지는 문제를 폴백 폰트 설정으로 해결하는 법, 그리고 Vim 스타일 분할 단축키까지.
---

Ghostty를 처음 설치하고 나서 영문은 깔끔한데 한국어가 깨지거나 두부 문자(□)로 나오는 경우가 있다. 원인은 단순하다. 지정한 폰트에 한글 글리프가 없기 때문이다. Ghostty는 이걸 `font-family`를 여러 번 선언해서 해결한다.

---

## 폰트 폴백 설정

Ghostty에서 `font-family`는 한 번만 쓰는 게 아니다. 여러 번 선언하면 **위에서 아래로 순서대로 폴백**된다. 앞에 있는 폰트에 해당 글리프가 없으면 다음 폰트에서 찾는다.

```
font-family = "JetBrainsMonoNL Nerd Font Mono"
font-family = "MesloLGS Nerd Font"
font-family = "NanumGothic"
```

- `JetBrainsMonoNL Nerd Font Mono` — 기본 영문 코딩 폰트. 대부분의 문자는 여기서 처리된다.
- `MesloLGS Nerd Font` — Nerd Font 아이콘 보강용.
- `NanumGothic` — 한글 폴백. 앞의 두 폰트에 없는 한국어 글리프를 여기서 가져온다.

세 번째 줄 하나만 추가하면 한국어 깨짐이 해결된다.

---

## 전체 설정

```
theme = Monokai Pro Octagon

font-family = "JetBrainsMonoNL Nerd Font Mono"
font-family = "MesloLGS Nerd Font"
font-family = "NanumGothic"

font-size = 14

keybind = ctrl+h=goto_split:left
keybind = ctrl+l=goto_split:right
keybind = ctrl+k=goto_split:up
keybind = ctrl+j=goto_split:down
```

설정 파일 위치는 `~/.config/ghostty/config`다.

---

## 단축키

### Vim 스타일 분할 이동

Ghostty는 터미널 분할(split)을 기본 지원한다. 기본 단축키는 방향키 기반인데, Vim을 쓴다면 `hjkl`로 바꾸는 게 자연스럽다.

```
keybind = ctrl+h=goto_split:left
keybind = ctrl+l=goto_split:right
keybind = ctrl+k=goto_split:up
keybind = ctrl+j=goto_split:down
```

### 기본 단축키 정리

분할 생성과 관리:

| 단축키            | 동작                         |
| ----------------- | ---------------------------- |
| `cmd+d`           | 수직 분할 (오른쪽에 새 패널) |
| `cmd+shift+d`     | 수평 분할 (아래에 새 패널)   |
| `cmd+w`           | 현재 분할 닫기               |
| `cmd+[` / `cmd+]` | 이전 / 다음 분할로 이동      |

탭:

| 단축키                        | 동작           |
| ----------------------------- | -------------- |
| `cmd+t`                       | 새 탭          |
| `cmd+shift+[` / `cmd+shift+]` | 이전 / 다음 탭 |
| `cmd+1` ~ `cmd+9`             | 탭 번호로 이동 |

기타:

| 단축키        | 동작           |
| ------------- | -------------- |
| `cmd+,`       | 설정 파일 열기 |
| `cmd+shift+,` | 설정 리로드    |
| `cmd+enter`   | 전체화면 토글  |
| `cmd+k`       | 터미널 지우기  |

---

## 폰트 설치 확인

NanumGothic이 없으면 폴백이 작동하지 않는다. macOS 기준:

```sh
brew install --cask font-nanum-gothic
```

설치 후 Ghostty를 재시작하면 한국어가 정상적으로 렌더링된다.
