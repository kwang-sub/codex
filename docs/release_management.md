# 릴리스 관리

현재 Codex 바이너리는 다음 세 곳에서 제공됩니다.

- GitHub Releases: https://github.com/openai/codex/releases/
- npm의 `@openai/codex`: https://www.npmjs.com/package/@openai/codex
- Homebrew의 `codex`: https://formulae.brew.sh/formula/codex

# 릴리스 만들기

새 릴리스를 게시하려면 리포지터리에서 `codex-rs/scripts/create_github_release` 스크립트를 실행하세요. 릴리스 유형에 따라 적절한 버전 번호가 자동으로 선택됩니다.

`main`에서 새로운 알파 릴리스를 만들려면(알파는 필요할 때 자주 만들어도 됩니다):

```
./codex-rs/scripts/create_github_release --publish-alpha
```

`main`에서 새로운 _공식_ 릴리스를 만들려면(더욱 주의가 필요합니다) 다음을 실행하세요.

```
./codex-rs/scripts/create_github_release --publish-release
```

TIP: `--dry-run` 플래그를 추가하면 해당 릴리스의 다음 버전 번호만 출력하고 종료합니다.

게시 스크립트를 실행하면 GitHub Actions가 릴리스 빌드를 시작합니다. https://github.com/openai/codex/actions/workflows/rust-release.yml 에서 해당 워크플로를 확인하세요. (참고: 향후 `gh`를 사용해 URL을 자동으로 찾도록 개선할 예정입니다.)

워크플로가 완료되면 GitHub Release는 "완료" 상태가 되지만, npm과 Homebrew를 따로 확인해야 합니다.

## npm 게시

npm 배포는 GitHub Action에서 처리합니다.

## Homebrew 게시

Homebrew는 자동화 시스템이 잘 갖춰져 있어 몇 시간마다 GitHub 리포지터리를 확인해 새 릴리스가 있는지 살펴봅니다. 새 릴리스를 발견하면 대응되는 Homebrew 릴리스를 만들기 위한 PR을 자동으로 올리며, 이는 다양한 macOS 버전에서 Codex CLI를 소스부터 빌드하는 작업을 포함합니다.

자동화 시스템이 릴리스를 감지했는지 확인하려면 아래 페이지를 주기적으로 새로고침해야 합니다.

https://github.com/Homebrew/homebrew-core/pulls?q=%3Apr+codex

모든 빌드가 완료되면 Homebrew 관리자가 PR을 승인해야 합니다. 전체 과정은 몇 시간이 걸리며 완전히 통제할 수는 없지만 대체로 원활하게 동작합니다.

참고로 Homebrew 포뮬라는 다음 위치에 있습니다.

https://github.com/Homebrew/homebrew-core/blob/main/Formula/c/codex.rb
