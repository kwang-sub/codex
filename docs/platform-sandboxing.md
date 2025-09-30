### 플랫폼 샌드박싱 상세 정보

Codex가 샌드박스 정책을 구현하는 방식은 운영체제에 따라 다릅니다.

- **macOS 12+**: **Apple Seatbelt**를 사용하며, 지정한 `--sandbox` 옵션에 대응하는 프로필(`-p`)로 `sandbox-exec`을 실행합니다.
- **Linux**: Landlock 및 seccomp API 조합을 사용해 `sandbox` 구성을 적용합니다.

Docker 등 컨테이너 환경에서 Linux를 실행할 때 호스트/컨테이너 설정이 필요한 Landlock/seccomp API를 지원하지 않으면 샌드박싱이 작동하지 않을 수 있습니다. 이 경우 원하는 수준의 샌드박스 보장을 제공하도록 Docker 컨테이너를 설정한 뒤, 컨테이너 내부에서 `--sandbox danger-full-access`(또는 보다 간단히 `--dangerously-bypass-approvals-and-sandbox`) 옵션과 함께 `codex`를 실행하는 것이 좋습니다.
