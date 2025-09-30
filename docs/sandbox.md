## 샌드박스 및 승인

### 승인 모드

Codex는 기본적으로 강력한 `Auto` 모드에서 동작합니다. 이 모드에서는 Codex가 작업 디렉터리 안에서 파일을 읽고, 수정하고, 명령을 자동으로 실행할 수 있습니다. 다만 작업 디렉터리 밖으로 나가거나 네트워크에 접근하려면 승인에 동의해야 합니다.

단순히 대화를 나누거나 본격적인 작업 전에 계획부터 세우고 싶다면 `/approvals` 명령으로 `Read Only` 모드로 전환하세요.

파일 읽기, 편집, 명령 실행, 네트워크 접근까지 모두 승인 없이 진행해야 한다면 `Full Access` 모드를 사용할 수 있습니다. 이 모드를 사용하기 전에는 충분히 주의하세요.

#### 기본값 및 권장 사항

- Codex는 기본적으로 강력한 가드레일이 있는 샌드박스에서 실행됩니다. 워크스페이스 밖의 파일 편집을 막고, 설정하지 않으면 네트워크 접근을 차단합니다.
- 실행 시 폴더가 버전 관리 중인지 감지하고 다음을 권장합니다.
  - 버전 관리되는 폴더: `Auto`(워크스페이스 쓰기 + on-request 승인)
  - 버전 관리되지 않은 폴더: `Read Only`
- 워크스페이스에는 현재 디렉터리와 `/tmp` 같은 임시 디렉터리가 포함됩니다. `/status` 명령으로 워크스페이스에 어떤 디렉터리가 포함되어 있는지 확인하세요.
- 직접 설정하려면 다음과 같이 실행할 수 있습니다.
  - `codex --sandbox workspace-write --ask-for-approval on-request`
  - `codex --sandbox read-only --ask-for-approval on-request`

### 승인을 전혀 받지 않고 실행할 수 있나요?

가능합니다. `--ask-for-approval never`를 사용하면 모든 승인 프롬프트가 비활성화됩니다. 이 옵션은 모든 `--sandbox` 모드와 함께 사용할 수 있으므로 Codex의 자율성 수준을 원하는 대로 제어할 수 있습니다. Codex는 주어진 제약 안에서 최선을 다합니다.

### 샌드박스 + 승인 조합 예시

| 목적 | 플래그 | 효과 |
| --- | --- | --- |
| 안전한 읽기 전용 탐색 | `--sandbox read-only --ask-for-approval on-request` | Codex가 파일을 읽고 질문에 답할 수 있습니다. 편집, 명령 실행, 네트워크 접근에는 승인이 필요합니다. |
| 읽기 전용 비대화형(CI) | `--sandbox read-only --ask-for-approval never` | 읽기만 수행하며 승인 요청이 없습니다. |
| 저장소 편집 허용, 위험 시 묻기 | `--sandbox workspace-write --ask-for-approval on-request` | 워크스페이스 안에서 파일 읽기/편집/명령 실행이 가능하며, 워크스페이스 밖이나 네트워크 접근 시 승인을 요청합니다. |
| Auto(프리셋) | `--full-auto` (즉 `--sandbox workspace-write` + `--ask-for-approval on-failure`) | 워크스페이스 안에서 읽기/편집/명령 실행이 가능하며, 샌드박스에서 명령이 실패하거나 권한 상승이 필요할 때 승인을 요청합니다. |
| YOLO(권장하지 않음) | `--dangerously-bypass-approvals-and-sandbox` (별칭: `--yolo`) | 샌드박스 없이 승인 프롬프트도 없습니다. |

> 참고: `workspace-write` 모드에서는 구성(`[sandbox_workspace_write].network_access = true`)으로 허용하지 않는 한 네트워크가 기본적으로 비활성화됩니다.

#### `config.toml`에서 세밀하게 조정하기

```toml
# 승인 모드
approval_policy = "untrusted"
sandbox_mode    = "read-only"

# full-auto 모드
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

# 선택 사항: workspace-write 모드에서 네트워크 허용
[sandbox_workspace_write]
network_access = true
```

**프로필**로 프리셋을 저장할 수도 있습니다.

```toml
[profiles.full_auto]
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

[profiles.readonly_quiet]
approval_policy = "never"
sandbox_mode    = "read-only"
```

### Codex 샌드박스 실험하기

Codex가 제공하는 샌드박스에서 명령을 실행하면 어떻게 동작하는지 테스트하려면 다음 하위 명령을 사용할 수 있습니다.

```
# macOS
codex debug seatbelt [--full-auto] [COMMAND]...

# Linux
codex debug landlock [--full-auto] [COMMAND]...
```

### 플랫폼별 샌드박스 상세

Codex가 샌드박스 정책을 구현하는 방식은 운영체제에 따라 다릅니다.

- **macOS 12+**: **Apple Seatbelt**를 사용하며 지정한 `--sandbox` 옵션에 대응하는 프로필(`-p`)로 `sandbox-exec`을 실행합니다.
- **Linux**: Landlock과 seccomp API 조합으로 `sandbox` 구성을 적용합니다.

Docker 같은 컨테이너 환경에서 Linux를 실행할 경우, 호스트/컨테이너 설정이 필요한 Landlock/seccomp API를 지원하지 않으면 샌드박싱이 작동하지 않을 수 있습니다. 이때는 원하는 수준의 보안을 제공하도록 Docker 컨테이너를 구성하고, 컨테이너 내부에서 `--sandbox danger-full-access`(또는 `--dangerously-bypass-approvals-and-sandbox`) 옵션으로 `codex`를 실행하는 것이 좋습니다.
