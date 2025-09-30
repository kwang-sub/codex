# 인증

## 사용량 기반 과금 대안: OpenAI API 키 사용

종량제로 이용하고 싶다면 OpenAI API 키로 계속 인증할 수 있습니다.

```shell
codex login --api-key "your-api-key-here"
```

이 키는 최소한 Responses API에 대한 쓰기 권한을 보유해야 합니다.

## API 키에서 ChatGPT 로그인으로 전환하기

이전에 Codex CLI를 API 키 기반 종량제 결제로 사용했다가 ChatGPT 요금제로 전환하려면 다음 단계를 따르세요.

1. CLI를 최신 버전으로 업데이트하고 `codex --version`이 `0.20.0` 이상인지 확인합니다.
2. `~/.codex/auth.json` 파일을 삭제합니다(Windows: `C:\\Users\\USERNAME\\.codex\\auth.json`).
3. `codex login`을 다시 실행합니다.

## "헤드리스" 머신에서 연결하기

현재 로그인 프로세스는 `localhost:1455`에서 서버를 실행합니다. Docker 컨테이너와 같은 "헤드리스" 서버나 원격 머신에 `ssh`로 접속한 경우, 로컬 머신의 브라우저에서 `localhost:1455`를 열어도 헤드리스 머신에서 실행 중인 웹 서버에 자동으로 연결되지 않습니다. 다음 우회 방법 중 하나를 사용하세요.

### 로컬에서 인증 후 "헤드리스" 머신으로 자격 증명 복사하기

가장 쉬운 방법은 웹 브라우저에서 `localhost:1455`에 접근할 수 있는 로컬 머신에서 `codex login` 프로세스를 완료하는 것입니다. 인증을 마치면 `$CODEX_HOME/auth.json` 파일이 생성됩니다(Mac/Linux에서는 `$CODEX_HOME` 기본 경로가 `~/.codex`, Windows에서는 `%USERPROFILE%\\.codex`).

`auth.json` 파일은 특정 호스트에 묶여 있지 않으므로 로컬에서 인증을 완료한 뒤 `$CODEX_HOME/auth.json` 파일을 헤드리스 머신으로 복사하면 해당 머신에서도 `codex`가 바로 동작합니다. Docker 컨테이너로 파일을 복사하려면 다음과 같이 실행할 수 있습니다.

```shell
# Docker 컨테이너 이름 또는 ID를 MY_CONTAINER로 바꿔서 사용하세요.
CONTAINER_HOME=$(docker exec MY_CONTAINER printenv HOME)
docker exec MY_CONTAINER mkdir -p "$CONTAINER_HOME/.codex"
docker cp auth.json MY_CONTAINER:"$CONTAINER_HOME/.codex/auth.json"
```

원격 머신에 `ssh`로 접속해 있다면 [`scp`](https://en.wikipedia.org/wiki/Secure_copy_protocol)를 사용하는 편이 좋습니다.

```shell
ssh user@remote 'mkdir -p ~/.codex'
scp ~/.codex/auth.json user@remote:~/.codex/auth.json
```

또는 아래와 같은 원라이너를 사용할 수도 있습니다.

```shell
ssh user@remote 'mkdir -p ~/.codex && cat > ~/.codex/auth.json' < ~/.codex/auth.json
```

### VPS 또는 원격 환경에서 포트 포워딩으로 연결하기

로컬 브라우저가 없는 원격 머신(VPS/서버)에서 Codex를 실행하면 로그인 도우미가 원격 호스트의 `localhost:1455`에서 서버를 시작합니다. 로컬 브라우저에서 로그인을 완료하려면 로그인 절차를 시작하기 전에 해당 포트를 로컬 머신으로 포워딩하세요.

```bash
# 로컬 머신에서 실행하세요.
ssh -L 1455:localhost:1455 <user>@<remote-host>
```

그 SSH 세션 안에서 `codex`를 실행하고 "Sign in with ChatGPT"를 선택합니다. 안내가 표시되면 출력된 URL(`http://localhost:1455/...`)을 로컬 브라우저에서 열면 됩니다. 트래픽이 원격 서버로 터널링됩니다.
