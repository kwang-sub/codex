## 설치 및 빌드

### 시스템 요구 사항

| 항목 | 세부 정보 |
| --- | --- |
| 운영체제 | macOS 12+, Ubuntu 20.04+/Debian 10+, 또는 Windows 11(WSL2를 통해) |
| Git(선택이지만 권장) | PR 도우미 내장 기능을 위해 2.23+ |
| RAM | 최소 4GB(권장 8GB) |

### DotSlash

GitHub Release에는 `codex`라는 이름의 [DotSlash](https://dotslash-cli.com/) 파일도 포함되어 있습니다. DotSlash 파일을 사용하면 개발 플랫폼과 관계없이 모든 기여자가 동일한 실행 파일 버전을 사용하도록 소스 제어에 가벼운 커밋을 남길 수 있습니다.

### 소스에서 빌드하기

```bash
# 리포지터리를 클론하고 Cargo 워크스페이스 루트로 이동합니다.
git clone https://github.com/openai/codex.git
cd codex/codex-rs

# Rust 툴체인이 없다면 설치합니다.
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustup component add rustfmt
rustup component add clippy

# Codex 빌드
cargo build

# 샘플 프롬프트로 TUI 실행
cargo run --bin codex -- "explain this codebase to me"

# 변경 후 코드를 정리합니다.
cargo fmt -- --config imports_granularity=Item
cargo clippy --tests

# 테스트 실행
cargo test
```
