## 고급 기능

## 비대화형 / CI 모드

Codex를 파이프라인에서 헤드리스로 실행할 수 있습니다. 아래는 GitHub Action 단계 예시입니다.

```yaml
- name: Update changelog via Codex
  run: |
    npm install -g @openai/codex
    codex login --api-key "${{ secrets.OPENAI_KEY }}"
    codex exec --full-auto "update CHANGELOG for next release"
```

### 비대화형 세션 이어서 실행하기

이전에 헤드리스로 실행한 세션을 다시 이어 받아 동일한 대화 문맥을 유지하고 같은 롤아웃 파일에 이벤트를 추가할 수 있습니다.

TUI에서 동일한 동작을 수행하려면 아래 명령을 사용합니다.

```shell
codex resume             # picker
codex resume --last      # most recent
codex resume <SESSION_ID>
```

호환성 정보:

- 최신 소스 빌드에는 `codex exec resume`이 포함되어 있습니다(아래 예시 참조).
- 현재 릴리스된 CLI에는 아직 포함되지 않았을 수 있습니다. `codex exec --help`에 `resume`이 표시되지 않으면 다음 소절의 우회 방법을 사용하세요.

```shell
# Resume the most recent recorded session and run with a new prompt (source builds)
codex exec "ship a release draft changelog" resume --last

# Alternatively, pass the prompt via stdin (source builds)
# Note: omit the trailing '-' to avoid it being parsed as a SESSION_ID
echo "ship a release draft changelog" | codex exec resume --last

# Or resume a specific session by id (UUID) (source builds)
codex exec resume 7f9f9a2e-1b3c-4c7a-9b0e-123456789abc "continue the task"
```

참고 사항:

- `--last`를 사용하면 Codex가 가장 최근에 기록된 세션을 선택합니다. 존재하지 않으면 새로 시작하는 것과 동일하게 동작합니다.
- 세션을 재개하면 기존 세션 파일에 새 이벤트가 추가되고 동일한 대화 ID가 유지됩니다.

## 추적 / 자세한 로깅

Codex는 Rust로 작성되었으므로 로깅 동작을 설정하기 위해 `RUST_LOG` 환경 변수를 지원합니다.

TUI의 기본값은 `RUST_LOG=codex_core=info,codex_tui=info`이며 로그 메시지는 `~/.codex/log/codex-tui.log`에 기록됩니다. 따라서 다른 터미널에서 아래 명령을 실행해 로그가 기록되는 즉시 확인할 수 있습니다.

```
tail -F ~/.codex/log/codex-tui.log
```

비대화형 모드(`codex exec`)는 기본적으로 `RUST_LOG=error`이며 메시지가 표준 출력에 바로 표시되므로 별도의 파일을 모니터링할 필요가 없습니다.

설정 옵션에 대한 자세한 내용은 Rust 문서의 [`RUST_LOG`](https://docs.rs/env_logger/latest/env_logger/#enabling-logging)를 참고하세요.

## Model Context Protocol(MCP)

Codex CLI는 `~/.codex/config.toml`에 [`mcp_servers`](./config.md#mcp_servers) 섹션을 정의하여 MCP 서버를 사용할 수 있도록 설정할 수 있습니다. Claude, Cursor와 같은 도구가 각자의 JSON 구성 파일에서 `mcpServers`를 정의하는 방식과 유사하지만, Codex는 JSON 대신 TOML을 사용하기 때문에 형식이 약간 다릅니다.

```toml
# IMPORTANT: the top-level key is `mcp_servers` rather than `mcpServers`.
[mcp_servers.server-name]
command = "npx"
args = ["-y", "mcp-server"]
env = { "API_KEY" = "value" }
```

## Codex를 MCP 서버로 사용하기

Codex CLI는 `codex mcp-server`를 통해 MCP _서버_로도 실행할 수 있습니다. 예를 들어 OpenAI [Agents SDK](https://platform.openai.com/docs/guides/agents)와 같은 멀티 에이전트 프레임워크 안에서 Codex를 도구로 활용하고 싶을 때 사용할 수 있습니다. MCP 서버 실행 구성을 추가/조회/가져오기/삭제하려면 `codex mcp` 명령을 별도로 사용하세요.

### Codex MCP 서버 빠른 시작

[Model Context Protocol Inspector](https://modelcontextprotocol.io/legacy/tools/inspector)를 사용해 Codex MCP 서버를 실행할 수 있습니다.

``` bash
npx @modelcontextprotocol/inspector codex mcp-server
```

`tools/list` 요청을 보내면 두 가지 도구를 사용할 수 있음을 확인할 수 있습니다.

**`codex`** – Codex 세션을 실행합니다. Codex 구성(struct)에 해당하는 설정 값을 받을 수 있습니다. `codex` 도구는 아래 속성을 사용합니다.

Property | Type | Description
---------|------|-------------
**`prompt`** (required) | string | Codex 대화를 시작할 때 사용할 초기 사용자 프롬프트입니다.
`approval-policy` | string | 모델이 생성한 셸 명령에 대한 승인 정책입니다. `untrusted`, `on-failure`, `never` 중 하나를 사용합니다.
`base-instructions` | string | 기본 안내문 대신 사용할 지침 집합입니다.
`config` | object | `$CODEX_HOME/config.toml`에 있는 값을 덮어쓸 개별 [구성 설정](https://github.com/openai/codex/blob/main/docs/config.md#config)입니다.
`cwd` | string | 세션의 작업 디렉터리입니다. 상대 경로인 경우 서버 프로세스의 현재 디렉터리를 기준으로 해석됩니다.
`include-plan-tool` | boolean | 대화에 계획 도구를 포함할지 여부입니다.
`model` | string | 모델 이름을 재정의할 때 사용합니다(예: `o3`, `o4-mini`).
`profile` | string | 기본 옵션을 지정하기 위한 `config.toml`의 구성 프로필입니다.
`sandbox` | string | 샌드박스 모드입니다. `read-only`, `workspace-write`, `danger-full-access` 중 하나를 사용합니다.

**`codex-reply`** – 대화 ID와 프롬프트를 제공해 Codex 세션을 이어갑니다. `codex-reply` 도구는 아래 속성을 사용합니다.

Property | Type | Description
---------|------|-------------
**`prompt`** (required) | string | Codex 대화를 계속하기 위한 다음 사용자 프롬프트입니다.
**`conversationId`** (required) | string | 이어서 진행할 대화의 ID입니다.

### 직접 사용해 보기

> [!TIP]
> Codex 실행에는 몇 분 정도가 걸리는 경우가 많습니다. 이를 고려해 MCP Inspector의 ⛭ Configuration에서 Request 및 Total timeout을 600000ms(10분)로 조정하세요.

아래 설정으로 MCP Inspector와 `codex mcp-server`를 사용해 간단한 틱택토 게임을 만들어 보세요.

**approval-policy:** never

**prompt:** Implement a simple tic-tac-toe game with HTML, Javascript, and CSS. Write the game in a single file called index.html.

**sandbox:** workspace-write

"Run Tool"을 클릭하면 Codex MCP 서버가 게임을 만드는 동안 발생하는 이벤트 목록이 표시됩니다.
