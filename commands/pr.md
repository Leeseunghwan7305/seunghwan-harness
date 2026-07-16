---
description: 변경을 커밋·push하고 GitHub PR을 생성하는 커맨드 (push/PR 전에 반드시 확인)
disable-model-invocation: true
---

변경을 커밋하고 push한 뒤 GitHub PR을 만들어줘. push와 PR 생성은 **되돌리기 어려운 외부 작업**이니, 실행 전에 반드시 승환님께 확인받아라.

`$ARGUMENTS` 에 지시가 있으면(예: 브랜치명, "draft", "제목 ...") 반영해라.

## 사전 점검

1. `gh auth status` 로 gh CLI 로그인 상태를 확인해라. 안 돼 있으면 "`gh auth login` 이 필요합니다" 라고 알리고 멈춰라.
2. `git status` 와 `git branch --show-current` 로 현재 상태와 브랜치를 확인해라.
3. `gh pr list --head <브랜치>` 로 이 브랜치에 **이미 열린 PR 이 있는지** 확인해라. 있으면 새로 만들지 말고 그 PR 을 알려준 뒤, push 만 할지 물어봐라.

## 브랜치 보호

- 지금 브랜치가 `main`(또는 `master`)이면 → **거기서 바로 PR 하지 마라.** 새 브랜치를 만들자고 제안해라. 브랜치명은 변경 성격에서 뽑아 `feat/...`, `fix/...` 처럼. 승환님이 정해주면 그걸로. 승인 후 `git switch -c <브랜치>`.
- 이미 feature 브랜치면 그대로 진행.

## 커밋

- 커밋 안 된 변경이 있으면: `/seunghwan-harness:commit-msg` 규칙(Conventional Commits, `<type>: <요약>`)으로 메시지를 만들어 보여주고, **커밋해도 될지 물어봐라.** 승인 후 커밋. 커밋 메시지 끝에 아래 줄을 붙여라:
  ```
  Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
  ```
- 이미 다 커밋돼 있으면 이 단계는 건너뛴다.
- 여러 성격이 섞였으면 `/seunghwan-harness:split-commit` 으로 나누는 게 나을지 짚어줘라.

## push & PR — 실행 전 확인

아래를 **한 번에 보여주고 승인받은 뒤** 실행해라:
- push 할 브랜치와 원격
- PR 제목: 대표 커밋 기반, Conventional Commits 스타일
- PR 본문 초안: 무엇을·왜 바꿨는지 요약 + 주요 변경 불릿 + 테스트/확인 방법. 끝에 아래 줄 추가:
  ```
  🤖 Generated with [Claude Code](https://claude.com/claude-code)
  ```

승인하면:
1. `git push -u origin <브랜치>`
2. `gh pr create --title "..." --body "..."` (draft 요청이면 `--draft`)
3. 생성된 **PR URL** 을 보여줘라.

승환님이 확인하기 전에는 **push 하거나 PR 을 만들지 마라.**
