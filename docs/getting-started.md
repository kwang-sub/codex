## 시작하기

### CLI 사용법

| 명령 | 목적 | 예시 |
| --- | --- | --- |
| `codex` | 대화형 TUI | `codex` |
| `codex "..."` | 대화형 TUI 시작 시 초기 프롬프트 제공 | `codex "fix lint errors"` |
| `codex exec "..."` | 비대화형 "자동화 모드" | `codex exec "explain utils.ts"` |

주요 플래그: `--model/-m`, `--ask-for-approval/-a`.

### 대화형 세션 이어서 실행하기

- `codex resume`을 실행해 세션 선택 UI를 표시합니다.
- 가장 최근 세션 재개: `codex resume --last`
- ID로 재개: `codex resume <SESSION_ID>`(세션 ID는 `/status` 또는 `~/.codex/sessions/`에서 확인할 수 있습니다.)

예시:

```shell
# 최근 세션 목록을 선택기로 열기
codex resume

# 가장 최근 세션 이어서 실행
codex resume --last

# 특정 세션 ID로 재개
codex resume 7f9f9a2e-1b3c-4c7a-9b0e-123456789abc
```

### 프롬프트를 입력으로 사용해 실행하기

프롬프트를 직접 입력해 Codex CLI를 실행할 수도 있습니다.

```shell
codex "explain this codebase to me"
```

```shell
codex --full-auto "create the fanciest todo-list app"
```

이렇게 하면 Codex가 파일을 스캐폴딩하고, 샌드박스에서 실행하며, 누락된 의존성을 설치하고, 라이브 결과를 보여줍니다. 변경 사항을 승인하면 작업 디렉터리에 반영됩니다.

### 프롬프트 예시

아래는 복사해 바로 사용할 수 있는 간단한 예시입니다. 따옴표 안의 문장을 원하는 작업으로 바꿔 사용하세요.

| ✨ | 입력 예시 | Codex가 수행하는 작업 |
| --- | --- | --- |
| 1 | `codex "Refactor the Dashboard component to React Hooks"` | 클래스 컴포넌트를 리팩터링하고 `npm test`를 실행한 뒤 변경 사항을 보여줍니다. |
| 2 | `codex "Generate SQL migrations for adding a users table"` | ORM을 추론하고 마이그레이션 파일을 만든 뒤 샌드박스된 DB에서 실행합니다. |
| 3 | `codex "Write unit tests for utils/date.ts"` | 테스트를 생성하고 실행하며 통과할 때까지 반복합니다. |
| 4 | `codex "Bulk-rename *.jpeg -> *.jpg with git mv"` | 파일 이름을 안전하게 변경하고 import/사용처를 업데이트합니다. |
| 5 | `codex "Explain what this regex does: ^(?=.*[A-Z]).{8,}$"` | 정규식을 단계별로 설명합니다. |
| 6 | `codex "Carefully review this repo, and propose 3 high impact well-scoped PRs"` | 현재 코드베이스에서 영향력 있는 PR 3개를 제안합니다. |
| 7 | `codex "Look for vulnerabilities and create a security review report"` | 보안 취약점을 찾아 설명합니다. |

### AGENTS.md로 메모리 제공하기

`AGENTS.md` 파일을 사용해 Codex에 추가 지침과 안내를 전달할 수 있습니다. Codex는 다음 위치에서 `AGENTS.md`를 찾아 위에서 아래로 병합합니다.

1. `~/.codex/AGENTS.md` – 개인용 전역 지침
2. 리포지터리 루트의 `AGENTS.md` – 프로젝트 공용 메모
3. 현재 작업 디렉터리의 `AGENTS.md` – 서브폴더/기능별 안내

AGENTS.md 활용 방법은 [공식 문서](https://agents.md/)를 참고하세요.

### 팁 & 바로가기

#### 파일 검색용 `@`

`@`를 입력하면 워크스페이스 루트에서 퍼지 파일명 검색이 실행됩니다. 위/아래 화살표로 결과를 선택하고 Tab 또는 Enter로 `@`을 선택한 경로로 교체하세요. Esc로 검색을 취소할 수 있습니다.

#### 이미지 입력

작성 창에 이미지를 붙여넣기(Ctrl+V / Cmd+V)하면 프롬프트에 첨부할 수 있습니다. CLI에서 `-i/--image`(쉼표로 구분)를 사용해 파일을 첨부할 수도 있습니다.

```bash
codex -i screenshot.png "Explain this error"
codex --image img1.png,img2.jpg "Summarize these diagrams"
```

#### Esc 두 번으로 이전 메시지 편집하기

채팅 입력란이 비어 있을 때 Esc를 누르면 “backtrack” 모드가 준비됩니다. Esc를 한 번 더 누르면 최근 사용자 메시지가 강조된 대화 미리보기가 열립니다. Esc를 계속 누르면 더 오래된 메시지로 이동합니다. Enter를 누르면 선택한 지점부터 대화를 분기하고, 해당 메시지를 입력 창에 채워 넣어 수정 후 다시 보낼 수 있습니다.

대화 미리보기 하단에는 편집이 활성화된 동안 `Esc edit prev` 힌트가 표시됩니다.

#### 셸 자동 완성

다음 명령으로 셸 완성 스크립트를 생성할 수 있습니다.

```shell
codex completion bash
codex completion zsh
codex completion fish
```

#### `--cd`/`-C` 플래그

Codex를 실행하기 전에 원하는 디렉터리로 직접 `cd`하는 것이 번거로울 때가 있습니다. 다행히 Codex는 `--cd` 옵션을 지원하므로 원하는 폴더를 지정할 수 있습니다. 새 세션이 시작될 때 TUI에 표시되는 **workdir**을 확인해 Codex가 `--cd` 값을 올바르게 사용하고 있는지 확인하세요.
