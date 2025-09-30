# 설정(Config)

Codex는 구성 값을 설정하기 위한 여러 방법을 지원합니다.

- `--model o3`와 같은 구성 전용 CLI 플래그(가장 높은 우선순위).
- `-c`/`--config` 플래그에 `key=value` 형태로 전달, 예: `--config model="o3"`.
  - 루트보다 더 깊은 값을 지정하려면 키에 점(`.`)을 포함할 수 있습니다. 예: `--config model_providers.openai.wire_api="chat"`.
  - `config.toml`과 일관성을 위해 값은 JSON이 아니라 TOML 형식의 문자열로 작성해야 합니다. 따라서 `key='{a = 1, b = 2}'`처럼 작성하고 `key='{"a": 1, "b": 2}'`는 사용하지 마세요.
    - 값 주위에 따옴표를 붙여야 셸이 공백을 기준으로 인자를 분리하지 않습니다. 따옴표가 없으면 `codex`는 `-c key={a`와 잘못된 추가 인자 `=`, `1,`, `b`, `=`, `2}`를 받게 됩니다.
  - 값은 TOML 객체라면 무엇이든 허용됩니다. 예: `--config shell_environment_policy.include_only='["PATH", "HOME", "USER"]'`.
  - `value`가 올바른 TOML 값으로 파싱되지 않으면 문자열로 처리됩니다. 따라서 `-c model='"o3"'`와 `-c model=o3`는 동일하게 동작합니다.
    - 첫 번째는 TOML 문자열 `"o3"`이고, 두 번째는 유효한 TOML이 아니므로 자동으로 문자열 `"o3"`로 취급됩니다.
    - 셸이 따옴표를 처리하므로 `-c key="true"`라고 입력하면 TOML에서는 `key = true`(불리언)로 해석됩니다. 문자열 `"true"`가 필요하다면 `-c key='"true"'`처럼 따옴표를 두 번 감싸야 합니다.
- `$CODEX_HOME/config.toml` 구성 파일. `CODEX_HOME` 환경 변수의 기본값은 `~/.codex`이며, 로그와 기타 Codex 관련 정보도 여기에 저장됩니다.

`--config` 플래그와 `config.toml` 파일은 아래 옵션을 지원합니다.

## model

Codex가 사용할 모델을 지정합니다.

```toml
model = "o3"  # 기본값 "gpt-5-codex"를 덮어씁니다.
```

## model_providers

Codex에 포함된 기본 모델 제공자 집합을 수정하거나 덮어쓸 수 있습니다. 이 값은 `model_provider` 옵션으로 선택할 수 있는 제공자 ID를 키로 사용하는 맵입니다.

예를 들어 OpenAI 4o 모델을 Chat Completions API로 사용하고 싶다면 다음과 같이 설정할 수 있습니다.

```toml
# TOML에서는 루트 키를 테이블보다 먼저 선언해야 합니다.
model = "gpt-4o"
model_provider = "openai-chat-completions"

[model_providers.openai-chat-completions]
# Codex UI에 표시될 이름
name = "OpenAI using Chat Completions"
# POST 요청을 보낼 기본 URL (`/chat/completions`가 뒤에 붙습니다)
base_url = "https://api.openai.com/v1"
# 이 제공자를 사용할 때 반드시 설정해야 하는 환경 변수 이름
# 값이 비어 있지 않아야 하며 HTTP 헤더의 `Bearer TOKEN`에 사용됩니다.
env_key = "OPENAI_API_KEY"
# wire_api는 "chat" 또는 "responses" 중 하나입니다. 생략하면 "chat"입니다.
wire_api = "chat"
# URL에 추가할 쿼리 매개변수(필요한 경우).
# 아래 Azure 예시를 참고하세요.
query_params = {}
```

이렇게 하면 OpenAI Chat Completions API와 호환되는 와이어 API를 사용하는 다른 모델로도 Codex CLI를 사용할 수 있습니다. 예를 들어 로컬에서 실행 중인 Ollama를 사용하려면 다음과 같이 정의할 수 있습니다.

```toml
[model_providers.ollama]
name = "Ollama"
base_url = "http://localhost:11434/v1"
```

다른 제공자(예: 별도의 환경 변수를 사용하는 API 키)도 다음처럼 설정할 수 있습니다.

```toml
[model_providers.mistral]
name = "Mistral"
base_url = "https://api.mistral.ai/v1"
env_key = "MISTRAL_API_KEY"
```

요청에 추가 HTTP 헤더가 필요하다면 하드코딩된 값(`http_headers`)이나 환경 변수로부터 읽은 값(`env_http_headers`)을 설정할 수 있습니다.

```toml
[model_providers.example]
# name, base_url 등 다른 필드도 함께 설정하세요.

# 모든 요청에 `X-Example-Header: example-value` 헤더를 추가합니다.
http_headers = { "X-Example-Header" = "example-value" }

# 환경 변수 `EXAMPLE_FEATURES`가 설정돼 있고 비어 있지 않다면
# 해당 값을 `X-Example-Features` 헤더로 추가합니다.
env_http_headers = { "X-Example-Features" = "EXAMPLE_FEATURES" }
```

### Azure 모델 제공자 예시

Azure는 `api-version`을 쿼리 매개변수로 반드시 전달해야 하므로 Azure 제공자를 정의할 때 `query_params`에 포함하세요.

```toml
[model_providers.azure]
name = "Azure"
# 자신의 서브도메인으로 바꿔야 합니다.
base_url = "https://YOUR_PROJECT_NAME.openai.azure.com/openai"
env_key = "AZURE_OPENAI_API_KEY"  # 또는 사용하는 값에 따라 "OPENAI_API_KEY"
query_params = { api-version = "2025-04-01-preview" }
wire_api = "responses"
```

Codex를 실행하기 전에 키를 내보내세요: `export AZURE_OPENAI_API_KEY=…`

### 제공자별 네트워크 튜닝

아래 선택적 설정은 **제공자별로** 재시도 동작과 스트리밍 유휴 타임아웃을 제어합니다. `config.toml`의 해당 `[model_providers.<id>]` 블록 안에 작성해야 합니다. (이전 버전에서 지원하던 최상위 키는 이제 무시됩니다.)

```toml
[model_providers.openai]
name = "OpenAI"
base_url = "https://api.openai.com/v1"
env_key = "OPENAI_API_KEY"
# 네트워크 튜닝 옵션 (모두 선택 사항이며 기본값으로 되돌아갑니다)
request_max_retries = 4            # 실패한 HTTP 요청 재시도 횟수
stream_max_retries = 10            # 끊어진 SSE 스트림 재시도 횟수
stream_idle_timeout_ms = 300000    # 스트리밍 유휴 타임아웃(5분)
```

#### request_max_retries

모델 제공자에 대한 HTTP 요청이 실패했을 때 Codex가 몇 번까지 재시도할지 결정합니다. 기본값은 `4`입니다.

#### stream_max_retries

스트리밍 응답이 중단되었을 때 Codex가 다시 연결을 시도하는 횟수입니다. 기본값은 `5`입니다.

#### stream_idle_timeout_ms

스트리밍 응답에서 활동이 없을 때 연결이 끊어졌다고 간주하기까지 기다리는 시간입니다. 기본값은 `300_000`(5분)입니다.

## model_provider

`model_providers` 맵에서 사용할 제공자를 지정합니다. 기본값은 `"openai"`입니다. 내장된 `openai` 제공자의 `base_url`은 `OPENAI_BASE_URL` 환경 변수로 재정의할 수 있습니다.

`model_provider`를 변경하면 일반적으로 `model`도 함께 덮어써야 합니다. 예를 들어 로컬에서 Mistral을 실행 중인 Ollama를 사용하려면 `model_providers`에 항목을 추가한 뒤 아래와 같이 설정해야 합니다.

```toml
model_provider = "ollama"
model = "mistral"
```

## approval_policy

Codex가 명령 실행 전에 사용자에게 승인을 요청하는 시점을 지정합니다.

```toml
# Codex는 미리 정의된 "신뢰되는" 명령 집합을 가지고 있습니다.
# approval_policy를 `untrusted`로 설정하면 그 집합에 속하지 않은 명령은 실행 전에 승인을 요청합니다.
#
# 사용자 지정 신뢰 명령을 지원하기 위한 계획은 https://github.com/openai/codex/issues/1260 에서 확인하세요.
approval_policy = "untrusted"
```

명령이 실패했을 때 알림을 받고 싶다면 `on-failure`를 사용하세요.

```toml
# 명령이 샌드박스에서 실패하면 Codex가 샌드박스 밖에서 다시 실행할지 물어봅니다.
approval_policy = "on-failure"
```

모델이 필요한 시점에만 권한 상승을 요청하도록 하려면 `on-request`를 사용하세요.

```toml
# 모델이 권한 상승 시점을 결정합니다.
approval_policy = "on-request"
```

또는 모델이 끝날 때까지 실행하게 하고 권한 상승 허용을 전혀 묻지 않으려면 `never`를 선택합니다.

```toml
# 사용자는 묻지 않습니다. 명령이 실패하면 Codex가 자동으로 다른 방법을 시도합니다.
# `exec` 하위 명령은 항상 이 모드를 사용합니다.
approval_policy = "never"
```

## profiles

_프로필_은 여러 구성 값을 묶어 한 번에 설정할 수 있는 모음입니다. `config.toml`에서 여러 프로필을 정의하고 실행 시 `--profile` 플래그로 사용할 프로필을 지정할 수 있습니다.

다음은 여러 프로필을 정의한 `config.toml` 예시입니다.

```toml
model = "o3"
approval_policy = "untrusted"

# `profile`을 설정하면 CLI에서 `--profile o3`를 지정한 것과 동일합니다.
# 물론 `--profile` 플래그로 이 값을 다시 덮어쓸 수도 있습니다.
profile = "o3"

[model_providers.openai-chat-completions]
name = "OpenAI using Chat Completions"
base_url = "https://api.openai.com/v1"
env_key = "OPENAI_API_KEY"
wire_api = "chat"

[profiles.o3]
model = "o3"
model_provider = "openai"
approval_policy = "never"
model_reasoning_effort = "high"
model_reasoning_summary = "detailed"

[profiles.gpt3]
model = "gpt-3.5-turbo"
model_provider = "openai-chat-completions"

[profiles.zdr]
model = "o3"
model_provider = "openai"
approval_policy = "on-failure"
```

사용자는 여러 수준에서 구성 값을 지정할 수 있으며, 우선순위는 다음과 같습니다.

1. 명령줄에서 지정한 개별 옵션(예: `--model o3`)
2. `--profile` 플래그로 지정한 프로필 값(또는 구성 파일에서 `profile`로 지정)
3. `config.toml`의 루트 값(예: `model = "o3"`)
4. Codex CLI의 기본값(예: 기본 모델은 `gpt-5-codex`)

## model_reasoning_effort

선택한 모델이 추론을 지원하는 것으로 알려져 있다면(예: `o3`, `o4-mini`, `codex-*`, `gpt-5`, `gpt-5-codex`) Responses API 사용 시 기본적으로 추론이 활성화됩니다. [OpenAI Platform 문서](https://platform.openai.com/docs/guides/reasoning?api-mode=responses#get-started-with-reasoning)에서 설명하듯 아래 값으로 설정할 수 있습니다.

- `"minimal"`
- `"low"`
- `"medium"`(기본값)
- `"high"`

추론을 최소화하려면 `"minimal"`을 선택하세요.

## model_reasoning_summary

모델 이름이 `"o"`로 시작(`"o3"`, `"o4-mini"` 등)하거나 `"codex"`로 시작하면 Responses API를 사용할 때 자동으로 추론이 활성화됩니다. [OpenAI Platform 문서](https://platform.openai.com/docs/guides/reasoning?api-mode=responses#reasoning-summaries)에 따르면 아래 값으로 설정할 수 있습니다.

- `"auto"`(기본값)
- `"concise"`
- `"detailed"`

추론 요약을 완전히 비활성화하려면 `model_reasoning_summary = "none"`으로 설정하세요.

```toml
model_reasoning_summary = "none"  # 추론 요약 비활성화
```

## model_verbosity

Responses API에서 GPT‑5 계열 모델의 출력 길이/상세 수준을 조정합니다. 사용할 수 있는 값은 다음과 같습니다.

- `"low"`
- `"medium"`(기본값)
- `"high"`

이 옵션을 설정하면 Codex가 요청 페이로드에 `"text": { "verbosity": "low" }`와 같이 구성된 값을 포함합니다.

예시:

```toml
model = "gpt-5"
model_verbosity = "low"
```

이 옵션은 Responses API를 사용하는 제공자에만 적용됩니다. Chat Completions 제공자에는 영향을 주지 않습니다.

## model_supports_reasoning_summaries

기본적으로 Codex는 추론 요약을 지원하는 것으로 알려진 OpenAI 모델에만 `reasoning` 값을 설정합니다. 현재 모델에 대해 강제로 `reasoning`을 활성화하려면 아래처럼 설정합니다.

```toml
model_supports_reasoning_summaries = true
```

## sandbox_mode

Codex는 모델이 생성한 셸 명령을 OS 수준 샌드박스 안에서 실행합니다.

대부분의 경우 아래 옵션만으로 원하는 동작을 선택할 수 있습니다.

```toml
# `--sandbox read-only`와 동일합니다.
sandbox_mode = "read-only"
```

기본 정책은 `read-only`입니다. 파일은 읽을 수 있지만 파일 쓰기나 네트워크 접근 시도가 차단됩니다.

좀 더 완화된 정책은 `workspace-write`입니다. 이 모드를 지정하면 Codex 작업의 현재 작업 디렉터리(및 macOS에서는 `$TMPDIR`)에 쓰기가 허용됩니다. CLI는 기본적으로 실행된 위치를 `cwd`로 사용하지만 `--cwd/-C` 옵션으로 변경할 수 있습니다.

macOS(그리고 곧 Linux)에서는 `.git/` 폴더가 _직접적인 하위 항목_인 쓰기 가능한 루트(예: `cwd`)의 경우 `.git/` 폴더만 읽기 전용으로 설정되고 저장소의 나머지는 쓰기가 허용됩니다. 따라서 기본적으로 `git commit` 같은 명령은 `.git/`에 쓰기를 시도하므로 실패하며, Codex가 권한 상승을 요청하게 됩니다.

```toml
# `--sandbox workspace-write`와 동일합니다.
sandbox_mode = "workspace-write"

# `sandbox = "workspace-write"`일 때만 적용되는 추가 설정입니다.
[sandbox_workspace_write]
# 기본적으로 Codex 세션의 cwd, $TMPDIR(설정된 경우), /tmp(존재하면)가 쓰기 가능입니다.
# 각 옵션을 true로 설정하면 해당 기본값을 비활성화합니다.
exclude_tmpdir_env_var = false
exclude_slash_tmp = false

# $TMPDIR와 /tmp 외에 추가로 쓰기 가능한 경로 목록(선택 사항)
writable_roots = ["/Users/YOU/.pyenv/shims"]

# 샌드박스 안에서 실행되는 명령이 네트워크에 접근할 수 있게 허용합니다.
# 기본값은 false입니다.
network_access = false
```

샌드박싱을 완전히 비활성화하려면 다음처럼 `danger-full-access`를 지정합니다.

```toml
# `--sandbox danger-full-access`와 동일합니다.
sandbox_mode = "danger-full-access"
```

Codex가 이미 자체 샌드박스를 제공하는 환경(예: Docker 컨테이너)에서 실행될 때 추가 샌드박싱이 불필요한 경우 유용합니다.

또는 오래된 Linux 커널이나 Windows처럼 Codex의 기본 샌드박스 메커니즘을 지원하지 않는 환경에서 Codex를 사용해야 할 때도 이 옵션이 필요할 수 있습니다.

## 승인 프리셋

Codex는 기본적으로 세 가지 승인 프리셋을 제공합니다.

- Read Only: 파일을 읽고 질문에 답할 수 있지만, 편집/명령 실행/네트워크 접근은 승인이 필요합니다.
- Auto: 워크스페이스 내에서는 파일 읽기, 편집, 명령 실행이 승인 없이 가능하며 워크스페이스 밖이나 네트워크 접근 시에만 승인을 요청합니다.
- Full Access: 디스크와 네트워크에 제한 없이 접근합니다. 매우 위험하므로 주의하세요.

추가로 `--ask-for-approval`과 `--sandbox` 옵션을 조합해 CLI에서 Codex 실행 방식을 세밀하게 조정할 수 있습니다.

## MCP 서버

Codex에 [MCP 서버](https://modelcontextprotocol.io/about)를 연결하면 [Playwright](https://github.com/microsoft/playwright-mcp), [Figma](https://www.figma.com/blog/design-context-everywhere-you-build/), [documentation](https://context7.com/) 등 외부 애플리케이션이나 리소스, 서비스를 사용할 수 있습니다. [공식 MCP 레지스트리](https://github.com/mcp?utm_source=blog-source&utm_campaign=mcp-registry-server-launch-2025)에서 더 많은 서버를 확인하세요.

### 서버 전송(transport) 설정

#### STDIO

```toml
# 최상위 테이블 이름은 반드시 `mcp_servers`여야 합니다.
# 하위 테이블 이름(`server-name` 부분)은 원하는 이름을 사용할 수 있습니다.
[mcp_servers.server-name]
command = "npx"
# 선택 사항
args = ["-y", "mcp-server"]
# 선택 사항: MCP 서버로 전달할 추가 환경 변수.
# Codex는 기본적으로 화이트리스트에 포함된 몇 가지 환경 변수만 전달합니다.
# https://github.com/openai/codex/blob/main/codex-rs/rmcp-client/src/utils.rs#L82
env = { "API_KEY" = "value" }
```

#### 스트리밍 HTTP

```toml
# 스트리밍 HTTP는 실험적 RMCP 클라이언트가 필요합니다.
experimental_use_rmcp_client = true
[mcp_servers.figma]
url = "http://127.0.0.1:3845/mcp"
# `Authorization: Bearer <token>` 헤더에 사용할 베어러 토큰(선택 사항)
# 토큰이 평문으로 저장되므로 주의해서 사용하세요.
bearer_token = "<token>"
```

### 기타 설정 옵션

```toml
# 선택 사항: 기본 10초 시작 타임아웃을 재정의합니다.
startup_timeout_sec = 20
# 선택 사항: 기본 60초 도구 타임아웃을 재정의합니다.
tool_timeout_sec = 30
```

### 실험적 RMCP 클라이언트

Codex는 [공식 Rust MCP SDK](https://github.com/modelcontextprotocol/rust-sdk)로 전환 중이며, 스트리밍 HTTP 서버와 같은 새로운 기능은 새 클라이언트에서만 동작합니다.

새 클라이언트를 사용해 보고 문제를 발견하면 알려주세요. 활성화하려면 `config.toml` 최상위에 아래를 추가합니다.

```toml
experimental_use_rmcp_client = true
```

### MCP CLI 명령

```shell
# 서버 추가 (env는 반복 가능, `--` 뒤에는 실행할 명령을 작성)
codex mcp add docs -- docs-server --port 4000

# 설정된 서버 목록 출력(표 또는 JSON)
codex mcp list
codex mcp list --json

# 특정 서버 정보 확인(표 또는 JSON)
codex mcp get docs
codex mcp get docs --json

# 서버 제거
codex mcp remove docs
```

## shell_environment_policy

Codex는 보조자가 제안한 `local_shell` 도구 호출을 실행할 때와 같이 서브프로세스를 생성합니다. 기본적으로는 **현재 환경 변수를 모두 그대로** 서브프로세스에 전달합니다. `config.toml`의 **`shell_environment_policy`** 블록에서 이 동작을 세밀하게 조정할 수 있습니다.

```toml
[shell_environment_policy]
# inherit에는 "all"(기본값), "core", "none"을 사용할 수 있습니다.
inherit = "core"
# 기본 필터(`"*KEY*"`, `"*TOKEN*"` 등)를 건너뛰려면 true로 설정하세요.
ignore_default_excludes = false
# 제외할 패턴(대소문자 구분 없이 glob 문법)
exclude = ["AWS_*", "AZURE_*"]
# 강제로 설정하거나 덮어쓸 값
set = { CI = "1" }
# 값을 지정하면 해당 패턴과 일치하는 변수만 유지됩니다.
include_only = ["PATH", "HOME"]
```

| 필드 | 타입 | 기본값 | 설명 |
| --- | --- | --- | --- |
| `inherit` | string | `all` | 환경 템플릿. `all`은 전체 환경을 복제하고, `core`는 `HOME`, `PATH`, `USER` 등 필수 값만 유지하며, `none`은 빈 환경으로 시작합니다. |
| `ignore_default_excludes` | boolean | `false` | `false`일 때 Codex는 다른 규칙을 적용하기 전에 **이름**에 `KEY`, `SECRET`, `TOKEN`이 포함된 변수를 제거합니다(대소문자 무시). |
| `exclude` | array<string> | `[]` | 기본 필터 이후에 제외할 패턴입니다. 예: `"AWS_*"`, `"AZURE_*"`. |
| `set` | table<string,string> | `{}` | 명시적으로 추가하거나 덮어쓸 값으로, 상속된 값보다 우선합니다. |
| `include_only` | array<string> | `[]` | 비어 있지 않다면 허용할 패턴 목록입니다. 패턴 중 하나와 일치하는 변수만 최종적으로 유지됩니다. 보통 `inherit = "all"`과 함께 사용합니다. |

패턴은 정규식이 아닌 **glob** 문법을 사용합니다. `*`는 임의 길이, `?`는 한 글자, `[A-Z]`나 `[^0-9]` 같은 문자 클래스를 지원합니다. 항상 대소문자를 구분하지 않습니다. 이 문법은 코드에서 `EnvironmentVariablePattern`으로 정의되어 있으며 `core/src/config_types.rs`에서 확인할 수 있습니다.

초기화된 환경을 깨끗하게 비우고 필요한 값만 설정하고 싶다면 다음과 같이 작성하세요.

```toml
[shell_environment_policy]
inherit = "none"
set = { PATH = "/usr/bin", MY_FLAG = "1" }
```

네트워크가 비활성화된 경우 `CODEX_SANDBOX_NETWORK_DISABLED=1` 환경 변수도 자동으로 설정되며, 현재는 이를 변경할 수 없습니다.

## otel

Codex는 실행 과정(외부 API 요청, 스트리밍 응답, 사용자 입력, 도구 승인 결정, 각 도구 호출 결과 등)을 설명하는 [OpenTelemetry](https://opentelemetry.io/) **로그 이벤트**를 내보낼 수 있습니다. 기본적으로 **비활성화**되어 있으므로 로컬 실행에는 아무 영향이 없습니다. 사용하려면 `[otel]` 테이블을 추가하고 내보낼 방식을 선택하세요.

```toml
[otel]
environment = "staging"   # 기본값은 "dev"
exporter = "none"          # 기본값은 "none". 이벤트를 전송하려면 otlp-http 또는 otlp-grpc로 설정하세요.
log_user_prompt = false    # 기본값 false. true로 설정하지 않으면 프롬프트 텍스트가 마스킹됩니다.
```

Codex는 모든 이벤트에 `service.name = $ORIGINATOR`(기본값 `codex_cli_rs`), CLI 버전, `env` 속성을 태그로 추가해 수집 시스템에서 개발/스테이징/프로덕션 트래픽을 구분할 수 있도록 합니다. `codex_otel` 크레이트에서 생성한 이벤트만 선택된 내보내기 대상으로 전달됩니다.

### 이벤트 카탈로그

모든 이벤트는 공통 메타데이터(`event.timestamp`, `conversation.id`, `app.version`, `auth_mode`, `user.account_id`, `terminal.type`, `model`, `slug`)를 공유합니다.

OTEL을 활성화하면 다음 이벤트 유형이 추가로 기록됩니다.

- `codex.conversation_starts`
  - `provider_name`
  - `reasoning_effort`(선택 사항)
  - `reasoning_summary`
  - `context_window`(선택 사항)
  - `max_output_tokens`(선택 사항)
  - `auto_compact_token_limit`(선택 사항)
  - `approval_policy`
  - `sandbox_policy`
  - `mcp_servers`(쉼표로 구분된 목록)
  - `active_profile`(선택 사항)
- `codex.api_request`
  - `attempt`
  - `duration_ms`
  - `http.response.status_code`(선택 사항)
  - `error.message`(실패 시)
- `codex.sse_event`
  - `event.kind`
  - `duration_ms`
  - `error.message`(실패 시)
  - `input_token_count`(응답일 때)
  - `output_token_count`(응답일 때)
  - `cached_token_count`(응답일 때, 선택 사항)
  - `reasoning_token_count`(응답일 때, 선택 사항)
  - `tool_token_count`(응답일 때)
- `codex.user_prompt`
  - `prompt_length`
  - `prompt`(기본적으로 마스킹되며, `log_user_prompt = true`일 때만 원문 포함)
- `codex.tool_decision`
  - `tool_name`
  - `call_id`
  - `decision` (`approved`, `approved_for_session`, `denied`, `abort`)
  - `source` (`config` 또는 `user`)
- `codex.tool_result`
  - `tool_name`
  - `call_id`(선택 사항)
  - `arguments`(선택 사항)
  - `duration_ms`(도구 실행 시간)
  - `success`(`"true"` 또는 `"false"`)
  - `output`

이벤트 형식은 향후 업데이트 과정에서 달라질 수 있습니다.

### 내보내기 대상 선택

이벤트 전송 위치는 `otel.exporter` 값으로 제어합니다.

- `none` – 계측은 유지하되 내보내지 않습니다(기본값).
- `otlp-http` – OTLP 로그 레코드를 OTLP/HTTP 수집기로 전송합니다. 수집기가 기대하는 엔드포인트, 프로토콜, 헤더를 지정하세요.

  ```toml
  [otel]
  exporter = { otlp-http = {
    endpoint = "https://otel.example.com/v1/logs",
    protocol = "binary",
    headers = { "x-otlp-api-key" = "${OTLP_TOKEN}" }
  }}
  ```

- `otlp-grpc` – OTLP 로그 레코드를 gRPC로 스트리밍합니다. 사용할 엔드포인트와 필요한 메타데이터 헤더를 지정하세요.

  ```toml
  [otel]
  exporter = { otlp-grpc = {
    endpoint = "https://otel.example.com:4317",
    headers = { "x-otlp-meta" = "abc123" }
  }}
  ```

exporter가 `none`이면 아무 곳에도 기록하지 않습니다. 그 외의 값을 사용한다면 직접 수집기를 실행하거나 기존 수집기 엔드포인트를 지정해야 합니다. 모든 exporter는 백그라운드 배치 워커에서 동작하며 Codex 종료 시 플러시됩니다.

소스에서 Codex를 빌드할 때는 OTEL 크레이트가 아직 `otel` 기능 플래그 뒤에 있습니다. 공식 사전 빌드 바이너리는 이 기능이 활성화된 상태로 제공됩니다. 기능 플래그를 비활성화하면 텔레메트리 훅이 무시되므로 추가 의존성 없이도 CLI는 정상 동작합니다.

## notify

Codex에서 발생한 이벤트를 알림으로 받고 싶다면 실행할 프로그램을 지정할 수 있습니다. Codex는 알림 인자를 JSON 문자열로 전달합니다. 예시는 다음과 같습니다.

```json
{
  "type": "agent-turn-complete",
  "turn-id": "12345",
  "input-messages": ["Rename `foo` to `bar` and update the callsites."],
  "last-assistant-message": "Rename complete and verified `cargo build` succeeds."
}
```

`"type"` 속성은 항상 포함됩니다. 현재는 `"agent-turn-complete"` 유형만 지원됩니다.

아래는 macOS에서 [terminal-notifier](https://github.com/julienXX/terminal-notifier)를 사용해 JSON을 파싱하고 푸시 알림 표시 여부를 결정하는 예시 Python 스크립트입니다.

```python
#!/usr/bin/env python3

import json
import subprocess
import sys


def main() -> int:
    if len(sys.argv) != 2:
        print("Usage: notify.py <NOTIFICATION_JSON>")
        return 1

    try:
        notification = json.loads(sys.argv[1])
    except json.JSONDecodeError:
        return 1

    match notification_type := notification.get("type"):
        case "agent-turn-complete":
            assistant_message = notification.get("last-assistant-message")
            if assistant_message:
                title = f"Codex: {assistant_message}"
            else:
                title = "Codex: Turn Complete!"
            input_messages = notification.get("input_messages", [])
            message = " ".join(input_messages)
            title += message
        case _:
            print(f"not sending a push notification for: {notification_type}")
            return 0

    subprocess.check_output(
        [
            "terminal-notifier",
            "-title",
            title,
            "-message",
            message,
            "-group",
            "codex",
            "-ignoreDnD",
            "-activate",
            "com.googlecode.iterm2",
        ]
    )

    return 0


if __name__ == "__main__":
    sys.exit(main())
```

이 스크립트를 알림용으로 사용하려면 컴퓨터에 있는 `notify.py` 경로에 맞춰 `~/.codex/config.toml`의 `notify` 항목을 설정하세요.

```toml
notify = ["python3", "/Users/mbolin/.codex/notify.py"]
```

> [!NOTE]
> 자동화나 다른 시스템과의 연동에는 `notify`를 사용하세요. Codex가 생성하는 각 이벤트마다 외부 프로그램을 실행해 단일 JSON 인자를 전달하며, TUI와는 독립적입니다. TUI 사용 중 가벼운 데스크톱 알림만 필요하다면 외부 프로그램이 필요 없는 `tui.notifications`를 사용하는 편이 낫습니다. 두 기능은 동시에 활성화할 수 있습니다. `tui.notifications`는 TUI 내 알림(예: 승인 요청)을 담당하고, `notify`는 시스템 레벨 훅이나 커스텀 알림에 적합합니다. 현재 `notify`는 `agent-turn-complete`만 내보내지만, `tui.notifications`는 `agent-turn-complete`과 `approval-requested`를 선택적으로 지원합니다.

## history

기본적으로 Codex CLI는 모델에 보낸 메시지를 `$CODEX_HOME/history.jsonl`에 기록합니다. UNIX 환경에서는 파일 권한이 `0600`으로 설정되어 소유자만 읽고 쓸 수 있습니다.

이 동작을 비활성화하려면 `[history]`를 다음과 같이 구성하세요.

```toml
[history]
persistence = "none"  # 기본값은 "save-all"
```

## file_opener

모델 출력에 포함된 인용을 하이퍼링크로 열 때 사용할 편집기/URI 스킴을 지정합니다. 값을 설정하면 터미널에서 ctrl/cmd+클릭으로 파일을 열 수 있도록 해당 스킴으로 변환됩니다.

예를 들어 모델 출력에 `【F:/home/user/project/main.py†L42-L50】`와 같은 참조가 있으면 `vscode://file/home/user/project/main.py:42` URI로 연결됩니다.

이 옵션은 `$EDITOR` 같은 일반 편집기 설정이 아니며, 아래 값만 허용합니다.

- `"vscode"`(기본값)
- `"vscode-insiders"`
- `"windsurf"`
- `"cursor"`
- `"none"` – 기능을 명시적으로 끕니다.

현재 기본값은 `"vscode"`지만 Codex는 VS Code 설치 여부를 확인하지 않습니다. 향후에는 `file_opener`가 `"none"`이나 다른 값으로 기본 설정될 수도 있습니다.

## hide_agent_reasoning

Codex는 최종 답변을 생성하기 전에 모델의 내부 "사고 과정"을 보여 주는 reasoning 이벤트를 간헐적으로 출력합니다. CI 로그나 간결한 터미널 출력에서는 이러한 이벤트가 방해될 수 있습니다.

`tui`와 `exec` 모드 모두에서 이 reasoning 이벤트를 숨기려면 `hide_agent_reasoning`을 `true`로 설정하세요.

```toml
hide_agent_reasoning = true   # 기본값은 false
```

## show_raw_agent_reasoning

가능한 경우 모델의 원시 chain-of-thought(“raw reasoning content”)를 표시합니다.

주의 사항:

- 선택한 모델/제공자가 실제로 raw reasoning을 제공할 때만 적용됩니다. 많은 모델은 해당 기능이 없습니다.
- 원시 reasoning에는 중간 생각이나 민감한 내용이 포함될 수 있습니다. 워크플로에 적합한 경우에만 활성화하세요.

예시:

```toml
show_raw_agent_reasoning = true  # 기본값은 false
```

## model_context_window

모델의 컨텍스트 윈도 크기(토큰 단위)입니다.

일반적으로 Codex는 가장 널리 사용되는 OpenAI 모델의 컨텍스트 윈도를 알고 있지만, 오래된 Codex CLI 버전에서 새로운 모델을 사용할 경우 이 값을 설정해 Codex가 남은 컨텍스트를 올바르게 계산하도록 할 수 있습니다.

## model_max_output_tokens

컨텍스트 윈도 설정과 유사하지만, 모델이 생성할 수 있는 최대 출력 토큰 수를 지정합니다.

## project_doc_max_bytes

세션 첫 턴에 포함할 `AGENTS.md` 내용을 읽을 때 사용할 최대 바이트 수입니다. 기본값은 32KiB입니다.

## tui

TUI에만 적용되는 옵션입니다.

```toml
[tui]
# 승인 요청이 필요하거나 턴이 완료될 때 데스크톱 알림을 보냅니다.
# 기본값은 false입니다.
notifications = true

# 특정 알림 유형만 수신하도록 선택할 수도 있습니다.
# 사용 가능한 값: "agent-turn-complete", "approval-requested"
notifications = [ "agent-turn-complete", "approval-requested" ]
```

> [!NOTE]
> Codex는 터미널 이스케이프 코드를 사용해 데스크톱 알림을 보냅니다. 모든 터미널이 이 기능을 지원하지 않습니다. (macOS Terminal.app과 VS Code 터미널은 커스텀 알림을 지원하지 않으며, iTerm2, Ghostty, WezTerm은 지원합니다.)

> [!NOTE]
> `tui.notifications`는 TUI 세션에 내장된 기능입니다. 여러 환경에서 프로그래밍 방식으로 알림을 받고 싶거나 운영체제 별 알림 시스템과 연동하려면 최상위 `notify` 옵션을 사용해 이벤트 JSON을 받는 외부 프로그램을 실행하세요. 두 설정은 서로 독립적이며 동시에 사용할 수 있습니다.

## 설정 요약표

| Key | Type / Values | Notes |
| --- | --- | --- |
| `model` | string | 사용할 모델(예: `gpt-5-codex`). |
| `model_provider` | string | `model_providers`의 제공자 ID(기본값: `openai`). |
| `model_context_window` | number | 컨텍스트 윈도 토큰 수. |
| `model_max_output_tokens` | number | 최대 출력 토큰 수. |
| `approval_policy` | `untrusted` \| `on-failure` \| `on-request` \| `never` | 승인 요청 시점. |
| `sandbox_mode` | `read-only` \| `workspace-write` \| `danger-full-access` | OS 샌드박스 정책. |
| `sandbox_workspace_write.writable_roots` | array<string> | workspace-write 모드에서 추가로 쓰기 가능한 경로. |
| `sandbox_workspace_write.network_access` | boolean | workspace-write 모드에서 네트워크 허용 여부(기본값: false). |
| `sandbox_workspace_write.exclude_tmpdir_env_var` | boolean | `$TMPDIR`를 쓰기 가능한 경로에서 제외(기본값: false). |
| `sandbox_workspace_write.exclude_slash_tmp` | boolean | `/tmp`를 쓰기 가능한 경로에서 제외(기본값: false). |
| `disable_response_storage` | boolean | ZDR 조직에서 필요. |
| `notify` | array<string> | 알림을 처리할 외부 프로그램. |
| `instructions` | string | 현재는 사용되지 않습니다. `experimental_instructions_file` 또는 `AGENTS.md`를 사용하세요. |
| `mcp_servers.<id>.command` | string | MCP 서버 실행 명령. |
| `mcp_servers.<id>.args` | array<string> | MCP 서버 명령 인자. |
| `mcp_servers.<id>.env` | map<string,string> | MCP 서버에 전달할 환경 변수. |
| `mcp_servers.<id>.startup_timeout_sec` | number | 서버 초기화 및 도구 초기 목록 조회에 걸리는 타임아웃(기본값: 10초). |
| `mcp_servers.<id>.tool_timeout_sec` | number | 도구 실행 타임아웃(기본값: 60초, 소수도 허용). |
| `model_providers.<id>.name` | string | 표시 이름. |
| `model_providers.<id>.base_url` | string | API 기본 URL. |
| `model_providers.<id>.env_key` | string | API 키에 사용할 환경 변수. |
| `model_providers.<id>.wire_api` | `chat` \| `responses` | 사용할 프로토콜(기본값: `chat`). |
| `model_providers.<id>.query_params` | map<string,string> | 추가 쿼리 매개변수(예: Azure `api-version`). |
| `model_providers.<id>.http_headers` | map<string,string> | 고정 헤더. |
| `model_providers.<id>.env_http_headers` | map<string,string> | 환경 변수에서 읽어오는 헤더. |
| `model_providers.<id>.request_max_retries` | number | 제공자별 HTTP 재시도 횟수(기본값: 4). |
| `model_providers.<id>.stream_max_retries` | number | SSE 재시도 횟수(기본값: 5). |
| `model_providers.<id>.stream_idle_timeout_ms` | number | SSE 유휴 타임아웃(ms)(기본값: 300000). |
| `project_doc_max_bytes` | number | `AGENTS.md`에서 읽을 최대 바이트 수. |
| `profile` | string | 활성 프로필 이름. |
| `profiles.<name>.*` | various | 프로필별 재정의 값. |
| `history.persistence` | `save-all` \| `none` | 히스토리 파일 보존 정책(기본값: `save-all`). |
| `history.max_bytes` | number | 현재는 사용되지 않습니다. |
| `file_opener` | `vscode` \| `vscode-insiders` \| `windsurf` \| `cursor` \| `none` | 클릭 가능한 인용을 열 때 사용할 URI 스킴(기본값: `vscode`). |
| `tui` | table | TUI 전용 옵션. |
| `tui.notifications` | boolean \| array<string> | TUI에서 데스크톱 알림 활성화(기본값: false). |
| `hide_agent_reasoning` | boolean | 모델 reasoning 이벤트 숨김. |
| `show_raw_agent_reasoning` | boolean | 원시 reasoning 표시(지원되는 경우). |
| `model_reasoning_effort` | `minimal` \| `low` \| `medium` \| `high` | Responses API 추론 강도. |
| `model_reasoning_summary` | `auto` \| `concise` \| `detailed` \| `none` | 추론 요약 모드. |
| `model_verbosity` | `low` \| `medium` \| `high` | GPT‑5 텍스트 자세함(Responses API). |
| `model_supports_reasoning_summaries` | boolean | 추론 요약 강제 활성화. |
| `model_reasoning_summary_format` | `none` \| `experimental` | 추론 요약 형식 강제 지정. |
| `chatgpt_base_url` | string | ChatGPT 인증 흐름용 기본 URL. |
| `experimental_resume` | string (path) | JSONL 재개 파일 경로(내부/실험적). |
| `experimental_instructions_file` | string (path) | 기본 지침을 대체할 파일(실험적). |
| `experimental_use_exec_command_tool` | boolean | 실험적 exec command 도구 사용. |
| `responses_originator_header_internal_override` | string | `originator` 헤더 값 덮어쓰기. |
| `projects.<path>.trust_level` | string | 프로젝트/워크트리를 신뢰함으로 표시(현재는 `"trusted"`만 인식). |
| `tools.web_search` | boolean | 웹 검색 도구 활성화(별칭: `web_search_request`, 기본값: false). |
