# seunghwan-harness

Claude Code 플러그인의 **네 가지 부품**(command·skill·agent·hook)을 각각 가장 작은 예제로 배우고, 그 위에 실제 쓰는 도구(git 워크플로우 + 다중 에이전트 팀)까지 얹은 플러그인입니다.

- **1~4장** — 네 부품을 최소 예제로 하나씩. **설치해서 직접 돌려보며 "아, 이건 이럴 때 도는구나"를 체감하는 것**이 목표입니다.
- **5장** — 그 위에 붙인 실전 커맨드 7개 (`review`·`commit-msg`·`split-commit`·`pr`·`team`·`pingpong` 등).

처음이면 위에서부터, 이미 감 잡았으면 [5장](#5-실전-커맨드-학습용-예제-너머)으로 바로 가세요.

---

## 1. 핵심: 네 부품은 "누가 발동시키는가"로 나뉜다

이 표 하나가 전부입니다. 나머지는 세부사항입니다.

| 부품 | 발동 주체 | 언제 도는가 | 이 저장소의 예제 |
|---|---|---|---|
| **command** | 사람 | 내가 `/이름` 을 직접 칠 때만 | `commands/hello.md` |
| **skill** | Claude | 대화가 `description` 과 맞으면 알아서 | `skills/explain-harness/SKILL.md` |
| **agent** | Claude | 별도 컨텍스트 창에 작업을 통째로 위임할 때 | `agents/reviewer.md` |
| **hook** | 하네스(프로그램) | 정해진 이벤트에 무조건. LLM 판단 개입 없음 | `hooks/hooks.json` |

### 제일 중요한 깨달음

**command와 skill은 사실 같은 것입니다.**

둘 다 마크다운 + frontmatter이고, 차이는 플래그 **하나**뿐입니다:

```yaml
disable-model-invocation: true    # 이게 있으면 → Claude가 자동으로 못 부름 → 사람 전용 = command
                                  # 이게 없으면 → Claude가 알아서 부름         = skill
```

`commands/hello.md` 와 `skills/explain-harness/SKILL.md` 를 나란히 열어보시면 이게 눈에 보입니다.

---

## 2. 설치

### 개발 중 (제일 빠른 방법)

설치 없이 바로 붙여서 테스트합니다:

```bash
claude --plugin-dir /Users/seunghwan/Desktop/seunghwan-harness
```

파일 고치고 나서는 세션 안에서 `/reload-plugins`.

### 어디서든 설치 (GitHub)

어느 컴퓨터에서든, 이 저장소를 clone하지 않고도:

```
/plugin marketplace add Leeseunghwan7305/seunghwan-harness
/plugin install seunghwan-harness@seunghwan-marketplace
```

`owner/repo` 만 적으면 Claude Code가 알아서 받아옵니다. 확인은 `/plugin list`.

**업데이트**: 이 저장소에 커밋을 push하면 그게 곧 새 버전입니다 (`version` 을 안 적어서 커밋 SHA가 버전이 되니까요). 받는 쪽에서는:

```
/plugin update seunghwan-harness@seunghwan-marketplace
```

### 로컬 경로로 설치

저장소가 이미 손에 있으면:

```
/plugin marketplace add /Users/seunghwan/Desktop/seunghwan-harness
/plugin install seunghwan-harness@seunghwan-marketplace
```

### 문법 검사

```bash
claude plugin validate /Users/seunghwan/Desktop/seunghwan-harness
```

`version` 이 없다는 경고가 하나 뜨는데 **의도한 것입니다** (아래 4번 참고). 그래서 경고를 에러로 취급하는 `--strict` 를 붙이면 이 저장소는 **실패합니다** — 정상입니다.

---

## 3. 실습 — 부품별로 직접 확인하기

### 부품 1: hook — 자동으로 돈다

**해볼 것**: 세션을 시작하고, 아무 `/` 도 치지 말고 그냥 물어봅니다.
```
세션 시작할 때 주입된 텍스트 중에 seunghwan-harness 들어간 줄 있어?
```

**나오는 것**: Claude가 이 줄을 인용해줍니다.
```
[seunghwan-harness] 플러그인 로드됨 — command / skill / agent / hook 4개 부품 대기중
```

**왜**: 제가 아무것도 안 시켰는데 이미 저 텍스트가 Claude 머릿속에 들어가 있습니다. hook은 **LLM이 아니라 프로그램이** 돌리기 때문에, Claude가 "이걸 실행할까?" 를 고민할 여지 자체가 없습니다. `SessionStart` 라는 순간이 오면 무조건 실행됩니다.

> **중요**: `SessionStart` hook의 `echo` 출력은 **터미널에 안 찍힙니다.** Claude의 **컨텍스트로 조용히 주입**됩니다. 그래서 위처럼 Claude에게 물어봐야 확인이 됩니다. (이벤트마다 stdout이 어디로 가는지가 다릅니다 — hook을 만들 때 제일 헷갈리는 부분입니다.)

### 부품 2: command — 사람이 부른다

**해볼 것**:
```
/seunghwan-harness:hello 승환
```

**나오는 것**: 인사 + 이 커맨드의 출처 설명

**왜**: `commands/hello.md` 본문이 그대로 Claude에게 프롬프트로 갔습니다. `$ARGUMENTS` 자리에 `승환` 이 들어갔고요. 커맨드 이름은 `플러그인이름:파일명` 규칙입니다.

이제 반대로 해보세요 — "hello 커맨드 좀 실행해줘" 라고 **말로** 부탁하면 Claude는 이걸 자동으로 못 씁니다. `disable-model-invocation: true` 때문입니다.

### 부품 3: skill — Claude가 알아서 부른다

**해볼 것**: `/` 없이 그냥 물어봅니다.
```
플러그인에서 hook이 뭐야?
```

**나오는 것**: `explain-harness` skill이 **스스로** 떠서 답변에 쓰입니다.

**왜**: Claude가 이 skill의 `description` 한 줄을 보고 "지금 상황에 맞네" 라고 판단했습니다. 본문은 그 판단 **후에야** 읽힙니다.

그래서 skill을 쓸 때 `description` 은 "무엇을 하는지" 가 아니라 **"언제 써야 하는지"** 를 써야 합니다. 이게 skill 설계의 90%입니다.

### 부품 4: agent — 통째로 위임한다

**해볼 것**:
```
reviewer 에이전트로 이 플러그인 설정 검토해줘
```

**나오는 것**: 파일 여러 개를 읽고 검토한 **결과 보고만** 돌아옵니다.

**왜**: agent는 **다른 컨텍스트 창**에서 돕니다. 파일을 10개 읽어도 내 창에는 최종 텍스트 한 덩어리만 옵니다 — 컨텍스트가 절약됩니다.

skill과의 차이가 여기입니다:
- **skill** = 지침이 **내 대화창 안으로** 끼어드는 것
- **agent** = **다른 창에서** 통째로 돌고 결과만 오는 것

`agents/reviewer.md` 의 `tools: [Read, Glob, Grep]` 도 보세요. 읽기 도구만 줘서 이 agent는 파일을 **고칠 수 없습니다**. 권한을 좁히는 게 agent의 큰 장점입니다.

---

## 4. 설정 파일 줄별 해설

> **JSON은 주석을 지원하지 않습니다.** `//` 를 넣으면 파싱이 깨져서 플러그인이 로드되지 않습니다. 그래서 해설을 여기에 씁니다.

### `.claude-plugin/plugin.json` — 플러그인 신분증

```json
{
  "name": "seunghwan-harness",
  "description": "...",
  "author": { "name": "seunghwan" }
}
```

- **`name`** — **유일한 필수 필드**입니다. kebab-case. 커맨드/skill 이름 앞에 붙는 네임스페이스가 됩니다 (`/seunghwan-harness:hello`).
- **`description`, `author`** — 전부 선택. `/plugin` 목록에서 보이는 용도.
- **`author.email` 이 없는 것도 의도적입니다.** 이것도 선택 필드인데, 이 저장소는 **public** 이라 적으면 이메일이 그대로 공개됩니다. 없어도 설치·동작에 아무 지장 없습니다.
- **`version` 이 없는 게 의도적입니다.** git 저장소에 있으면 **커밋 SHA가 곧 버전**이 됩니다. 즉 커밋할 때마다 자동으로 "새 버전"이 되어서, 개발 중엔 버전 숫자를 손으로 올릴 필요가 없습니다.
- **`commands`, `skills`, `agents`, `hooks` 경로를 안 적은 것도 의도적입니다.** 이건 전부 **기본 위치**라서 자동으로 발견됩니다. 경로를 적는 건 기본 위치를 벗어날 때만 필요합니다.

### `.claude-plugin/marketplace.json` — "어디서 받는지"

`plugin.json` 이 "이게 뭔지" 라면, 이건 "어디서 받는지" 입니다. 이 파일이 있어야 `/plugin marketplace add` 가 됩니다.

```json
{
  "name": "seunghwan-marketplace",
  "owner": { "name": "seunghwan" },
  "plugins": [
    { "name": "seunghwan-harness", "source": "./" }
  ]
}
```

- **`name`, `owner.name`, `plugins`** — 필수 3종.
- **`plugins[].name`** — `plugin.json` 의 `name` 과 **일치해야** 합니다.
- **`source: "./"`** — 이 저장소가 **마켓플레이스이자 플러그인**이라는 뜻. 상대 경로는 `.claude-plugin/` 이 아니라 **저장소 루트 기준**으로 풀립니다.
- 마켓플레이스 하나에 플러그인을 여러 개 담을 수도 있습니다. `plugins` 가 배열인 이유입니다.

### `hooks/hooks.json` — 언제 무엇을 돌릴지

```json
{
  "hooks": {
    "SessionStart": [
      { "hooks": [ { "type": "command", "command": "echo '...'" } ] }
    ]
  }
}
```

중첩이 좀 헷갈립니다. 읽는 법:

- 바깥 **`"hooks"`** — 최상위 껍데기. 고정.
- **`"SessionStart"`** — **이벤트 이름**. 이게 발동 시점입니다.
- 그 안의 **배열** — 이 이벤트에 여러 그룹을 걸 수 있어서 배열. (여기에 `"matcher"` 를 넣어 조건을 걸 수 있습니다. 지금은 조건 없음 = 항상.)
- 안쪽 **`"hooks"`** — 실제로 실행할 것들의 목록.
- **`"type": "command"`** — 쉘 명령을 돌린다는 뜻.

---

## 5. 실전 커맨드 (학습용 예제 너머)

4부품을 배우고 나서 실제로 쓰려고 붙인 커맨드 7개입니다. 크게 **git 워크플로우**와 **다중 에이전트** 두 갈래입니다.

전부 사람 전용(`disable-model-invocation: true`)이라 `/` 로 직접 불러야 돌고, 되돌리기 어려운 작업(커밋·push·PR)은 **실행 전에 반드시 확인**을 받습니다.

### git 워크플로우 — 작업 → 점검 → 커밋 → PR

```
작업 → /review → /commit-msg 또는 /split-commit → /pr
```

| 커맨드 | 하는 일 |
|---|---|
| `/seunghwan-harness:review` | 지금 고친 변경(working diff)을 `code-reviewer` 에이전트로 **커밋 전 셀프 점검**. `review staged` 로 staged만, 경로를 주면 그 파일만 |
| `/seunghwan-harness:commit-msg` | staged된 변경을 보고 커밋 메시지 초안 작성. **커밋은 안 하고 초안만** |
| `/seunghwan-harness:split-commit` | 이미 작업한 변경을 **기능 단위로 나눠** 여러 커밋으로. 계획을 먼저 보여주고 승인 후 커밋. push는 안 함 |
| `/seunghwan-harness:pr` | 커밋·push·GitHub PR 생성까지. push/PR 전 확인, `main`/`master` 에선 새 브랜치부터 만들도록 보호 |

커밋 메시지는 **Conventional Commits** 형식(`<type>: <요약>`)으로 만듭니다:
`feat`(새 기능) · `fix`(버그) · `hotfix`(긴급 수정) · `refactor`(구조 개선) · `docs` · `test` · `chore` · `style` · `perf`. **스코프는 쓰지 않습니다** — 항상 `feat: <내용>` 형식으로만 (`feat(team): ...` 같은 괄호 스코프 금지).

### 다중 에이전트 — 역할을 나눠 일 시키기

**`/seunghwan-harness:team <작업>`** — 하나의 작업을 **역할별 팀**에게 단계별로 넘겨 계획을 세웁니다.

```
/seunghwan-harness:team 로그인 페이지에 비밀번호 찾기 추가
/seunghwan-harness:team --all 결제 모듈 리팩터링       # 전원 소집
```

팀은 10개 역할 에이전트로 구성됩니다:

| 단계 | 역할 |
|---|---|
| 이해 | `pm` (요구사항) · `researcher` (조사) |
| 설계 | `architect` (구조) |
| 구현 계획 | `frontend` · `backend` |
| 검증 | `code-reviewer` · `qa` · `perf` · `security` |
| 문서 | `docs` |

**`/seunghwan-harness:pingpong <작업>`** — 개발자와 리뷰어를 **리뷰어가 OK 할 때까지 주거니 받거니** 반복시켜 계획을 다듬습니다. 개발자는 이전 라운드를 기억한 채 고치고, 최대 라운드(기본 3, `... 5라운드` 로 조절)에 도달하면 멈춥니다.

```
🏓 라운드 1 → 개발자 계획 / 리뷰어 "막아야 함 2개"
🏓 라운드 2 → 개발자 수정 / 리뷰어 "없음"  →  ✅ 종료
```

**다중 에이전트 커맨드의 핵심 설계 두 가지:**
- 모든 역할이 `tools: [Read, Glob, Grep]` 만 가집니다 — **파일을 못 고칩니다.** 그래서 "계획까지만, 실행은 승인 후"가 구조적으로 보장됩니다.
- `team` 은 매번 10명을 다 부르지 않습니다. **작업에 맞는 역할만** 골라 부르고, 누구를 왜 뺐는지 밝힙니다. 전원 소집은 `--all`.

> 이 커맨드들이 **커맨드 + 에이전트 + (역할 위임)** 을 조합한 실제 예입니다. 어떻게 맞물리는지 코드로 보고 싶으면 `commands/team.md`·`commands/pingpong.md` 와 `agents/*.md` 를 열어보세요.

> 참고: `agents/reviewer.md` 는 위 팀과 별개인 **플러그인 설정 검사용** 에이전트입니다(부품 3 학습 예제). 팀의 코드 리뷰어는 `agents/code-reviewer.md` 입니다.

---

## 6. 사용 예시 — commit-msg 직접 돌려보기

`commit-msg` 가 뭘 하는지는 위 5번에서 설명했습니다. 여기서는 그걸 **실제로 눌러서 뭐가 나오는지** 봅니다.

**1) 재현용 변경 하나를 stage 합니다** (실제 작업 중이면 그 변경을 그대로 써도 됩니다):

```bash
echo "# 사용 예시 테스트" >> README.md
git add README.md
```

**2) 커맨드를 칩니다:**

```
/seunghwan-harness:commit-msg
```

**3) 나오는 것** — 문구는 변경 내용마다 다르지만, 아래 **형태**는 항상 같습니다. 커밋 메시지 초안과, 바로 쓸 수 있는 `git commit` 명령이 함께 나옵니다:

> **README에 테스트 줄 추가**
>
> \- 사용 예시 동작 확인을 위한 임시 변경
>
> 바로 커밋하려면: `git commit -m "README에 테스트 줄 추가"`

**확인 포인트**: `git status` 를 다시 보면 여전히 staged 상태이고 **아직 커밋되지 않았습니다.** 이 커맨드는 초안만 보여주고 커밋은 하지 않기 때문입니다 (`commands/commit-msg.md` 참고). 커밋하려면 나온 `git commit` 줄을 직접 실행하거나 "커밋해줘" 라고 요청해야 합니다.

**원상복구** (테스트로만 써봤다면):

```bash
git reset README.md && git checkout README.md
```

> 이게 섹션 3(부품별 개념 확인)과 다른 점입니다. 3번은 "왜 도는가"를 이해시키고, 여기는 실전 커맨드 하나를 **손으로 끝까지 돌려 결과를 눈으로 확인**시킵니다.

---

## 7. 자주 틀리는 것

**`.claude-plugin/` 안에는 `plugin.json` 과 `marketplace.json` 만 넣습니다.**

`commands/`, `skills/`, `agents/`, `hooks/` 를 `.claude-plugin/` 안에 넣으면 **발견되지 않습니다**. 전부 저장소 루트에 둬야 합니다.

```
seunghwan-harness/
├── .claude-plugin/          ← 여기엔 json 2개만!
│   ├── plugin.json
│   └── marketplace.json
├── commands/                ← 루트에
├── skills/                  ← 루트에
├── agents/                  ← 루트에
├── hooks/                   ← 루트에
└── README.md
```

기타:
- skill 파일명은 **정확히 `SKILL.md`** (대문자). `skills/이름/SKILL.md` 구조.
- JSON에 주석 금지. 마지막 쉼표(trailing comma)도 금지.
- 파일 고쳤는데 반영이 안 되면 `/reload-plugins`.

---

## 8. 다음에 해볼 것

**다른 hook 이벤트 써보기** — `SessionStart` 말고도 많습니다:

`UserPromptSubmit` (내가 뭘 칠 때마다) · `PreToolUse` (도구 쓰기 직전 — 여기서 막을 수도 있음) · `PostToolUse` (도구 쓴 직후 — 자동 포맷팅 같은 것) · `Stop` (Claude가 답 끝냈을 때) · `SessionEnd`

> ⚠️ `PreToolUse` / `PostToolUse` 는 파일을 고치거나 도구를 차단할 수 있어서 실수하면 아플 수 있습니다. 이 저장소가 제일 안전한 `SessionStart` 하나만 쓴 이유입니다. 건드릴 땐 작게 시작하세요.

**변수 써보기** — hook 커맨드나 MCP 설정 안에서:
- `${CLAUDE_PLUGIN_ROOT}` — 플러그인이 설치된 경로. **업데이트하면 바뀝니다.**
- `${CLAUDE_PLUGIN_DATA}` — 업데이트해도 **살아남는** 저장 공간. 캐시나 상태를 여기 두세요.
- `${CLAUDE_PROJECT_DIR}` — 지금 프로젝트 루트.

쉘 명령에 쓸 땐 따옴표로 감싸세요: `"\"${CLAUDE_PLUGIN_ROOT}\"/scripts/foo.sh"`

**더 있는 부품들** — 4부품에 집중하려고 뺐지만, 플러그인은 이것도 담을 수 있습니다: MCP 서버(`.mcp.json`) · LSP 서버(`.lsp.json`) · output styles · `userConfig` (설치할 때 API 키 같은 걸 물어보기)

**공식 문서**: https://code.claude.com/docs/en/plugins

---

## 9. 세션을 넘는 기억 — 부결 캐시

지금까지의 커맨드는 전부 **매번 백지에서 시작**합니다. `/team`·`/pingpong`이 지난 세션에 "이 접근은 이래서 기각"이라고 결론 내도, 다음 세션의 에이전트는 그걸 모르고 같은 접근을 또 제안하죠. 8장이 예고한 "**살아남는 저장 공간**"을 여기서 실제로 씁니다.

**어디에 저장하나** — `~/.claude/seunghwan-harness/rejected.jsonl`

8장은 `${CLAUDE_PLUGIN_DATA}` 를 권했지만, 이 변수는 **hook 커맨드 문자열과 MCP 설정에서만 확장**됩니다. 커맨드 본문에서 Claude가 Bash로 직접 쓸 때는 안 풀려서(빈 문자열), 대신 `~/.claude/` 밑에 둡니다. 목표는 똑같이 충족됩니다 — **플러그인을 업데이트해도 살아남고**(홈 디렉터리라서), **git 저장소 바깥**이라 커밋되지 않는 per-user 런타임 상태입니다.

**형식** — append-only JSONL, 한 줄이 부결 하나:

```json
{"date":"2026-07-17","cmd":"team","task":"로그인 비밀번호 찾기","approach":"이메일 링크 대신 SMS OTP","reason":"SMS 비용+외부 의존성, 보안 리뷰어 지적","by":"user"}
```

**언제 읽고 언제 쓰나**

- **읽기(시작)** — `/team`·`/pingpong`이 일을 시작하기 전에 이 파일을 `cat` 해서, 이번 작업과 **유형이 겹치는** 과거 부결만 골라 에이전트에게 넘깁니다.
- **쓰기(최종 기각 시)** — 접근이 **최종적으로 버려졌을 때만** 한 줄 append 합니다. 핑퐁 도중 고쳐진 "막아야 함"은 **기록하지 않습니다** — 그건 해결된 거지 폐기가 아니니까요.
- **상한** — 쓸 때마다 `tail -n 200` 으로 **최근 200개만** 남깁니다. 무한정 커지지 않고, 오래된 부결은 자연히 밀려납니다 (내장 `MEMORY.md` 의 ~200줄 상한과 같은 발상).

**중요: 하드 블록이 아닙니다.** 과거 부결은 "참고"로만 주입됩니다. 조건이 달라졌으면 그 접근을 다시 제안해도 됩니다 — 단 **왜 이번엔 다른지** 밝혀야 합니다. (조건 A에서 기각된 게 조건 B에선 맞을 수 있으니, 무조건 막으면 오히려 손해입니다.)

> 이게 **hook 없이** 커맨드 본문의 Bash 지시만으로 상태를 남기는 예입니다. "무엇이 진짜 기각인가"는 판단이 필요해서, 프로그램(hook)이 아니라 **Claude가 주도**하는 게 맞습니다. 읽기/쓰기 시점은 `commands/team.md`·`commands/pingpong.md` 를 열어보세요.
