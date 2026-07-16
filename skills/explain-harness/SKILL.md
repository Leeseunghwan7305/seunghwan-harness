---
name: explain-harness
description: Claude Code 플러그인의 부품(command, skill, agent, hook)이 각각 무엇이고 언제 실행되는지 물어볼 때 사용
---

승환님이 플러그인 부품에 대해 물으셨습니다. 아래 내용을 바탕으로, 질문한 부분에 집중해서 한국어로 설명해주세요.

## 핵심: 네 부품은 "누가 발동시키는가"로 나뉩니다

| 부품 | 발동 주체 | 언제 도는가 |
|---|---|---|
| **command** | 사람 | 사용자가 `/이름` 을 직접 칠 때만 |
| **skill** | Claude | 대화 내용이 skill의 `description` 과 맞으면 알아서 |
| **agent** | Claude | 별도 컨텍스트 창에 하위 작업을 통째로 위임할 때 |
| **hook** | 하네스(프로그램) | 정해진 이벤트에 무조건. LLM 판단이 개입하지 않음 |

## 부품별 상세

**command** (`commands/이름.md`)
마크다운 파일 하나. 본문이 곧 Claude에게 가는 프롬프트입니다. `$ARGUMENTS` 로 사용자가 뒤에 붙인 입력을 받습니다. 파일명이 커맨드 이름이 됩니다.

**skill** (`skills/이름/SKILL.md`)
frontmatter의 `description` 이 **유일하게 중요한 필드**입니다. Claude는 이 한 줄만 보고 "지금 이 skill을 불러야 하나?" 를 판단합니다. 본문은 불린 다음에야 읽힙니다. 그래서 description은 "무엇을 하는지" 보다 **"언제 써야 하는지"** 를 써야 합니다.

**command와 skill은 사실 같은 것입니다.** 둘 다 마크다운 + frontmatter이고, 유일한 차이는 `disable-model-invocation: true` 플래그입니다. 이게 있으면 Claude가 자동으로 못 부르니 사람 전용(= command)이 되고, 없으면 Claude가 알아서 부릅니다(= skill).

**agent** (`agents/이름.md`)
skill은 지침이 **내 대화창 안으로** 끼어드는 것이고, agent는 **다른 창에서 통째로 돌고 결과 텍스트만** 돌아오는 것입니다. 그래서 agent는 컨텍스트를 아껴줍니다 — 파일 20개를 읽어도 내 창에는 요약 한 문단만 옵니다. `tools:` 로 쓸 수 있는 도구를 좁힐 수 있습니다.

**hook** (`hooks/hooks.json`)
유일하게 **LLM이 아닌 프로그램이 돌리는** 부품입니다. `SessionStart`, `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop` 같은 이벤트 이름에 명령을 걸어두면, 그 순간이 오면 Claude의 판단과 무관하게 무조건 실행됩니다. "매번 반드시" 가 필요하면 hook, "상황 봐서" 면 skill입니다.

## 파일이 놓이는 위치

`.claude-plugin/` 안에는 `plugin.json` 과 `marketplace.json` **만** 들어갑니다. `commands/`, `skills/`, `agents/`, `hooks/` 는 전부 플러그인 루트에 둡니다. 여기가 제일 흔히 틀리는 지점입니다.

## 설명 후

승환님이 지금 `seunghwan-harness` 플러그인을 학습 중이라면, 해당 부품의 실제 예제 파일 경로를 알려주고 열어보시라고 안내해주세요.
