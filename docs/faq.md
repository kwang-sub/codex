## 자주 묻는 질문(FAQ)

### 2021년에 공개된 OpenAI Codex 모델과 관련이 있나요?

2021년 OpenAI는 자연어 프롬프트로 코드를 생성하는 AI 시스템 Codex를 출시했습니다. 그 원래 Codex 모델은 2023년 3월에 사용 중단되었고, 현재 CLI 도구와는 별개의 제품입니다.

### 어떤 모델을 지원하나요?

Codex는 최고의 코딩 모델인 GPT-5와 함께 사용하는 것을 권장합니다. 기본 추론 수준은 medium이며, 복잡한 작업에는 `/model` 명령으로 high로 올릴 수 있습니다.

API 기반 인증과 `--model` 플래그를 사용하면 이전 모델도 사용할 수 있습니다.

### 왜 `o3`나 `o4-mini`가 작동하지 않나요?

API에서 스트리밍 응답과 chain-of-thought 요약을 받으려면 [API 계정을 인증](https://help.openai.com/en/articles/10910291-api-organization-verification)해야 할 수 있습니다. 그래도 문제가 계속되면 알려주세요!

### Codex가 파일을 수정하지 못하게 하려면 어떻게 하나요?

기본적으로 Codex는 현재 작업 디렉터리에서 파일을 수정할 수 있습니다(Auto 모드). 편집을 막으려면 CLI 플래그 `--sandbox read-only`를 사용해 읽기 전용 모드로 실행하세요. 또는 대화 중간에 `/approvals` 명령으로 승인 수준을 변경할 수 있습니다.

### Windows에서도 사용할 수 있나요?

Windows에서 직접 Codex를 실행할 수도 있지만 공식적으로 지원되지는 않습니다. [Windows Subsystem for Linux(WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install) 사용을 권장합니다.
