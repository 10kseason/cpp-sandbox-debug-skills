# cpp-sandbox-debug-skills

Codex용 C++ 샌드박스 디버그 스킬입니다.

GUI 디버거를 쓰기 어렵거나, 프로세스 attach/외부 트레이싱 도구가 제한된 환경에서 C++ 프로젝트를 더 체계적으로 디버깅하도록 돕는 워크플로 스킬입니다. 크래시, 파서 실패, 런치 실패, 설정 불일치, race condition, 타이밍 문제, 오디오/렌더 글리치 같은 이슈를 `재현 -> 국소화 -> 좁은 계측 -> 결정적 재현 -> 보수적 수정 -> 검증` 흐름으로 다루는 데 초점을 둡니다.

## What This Skill Is Good At

- 로컬 C++ 저장소에서 재현 가능한 버그 디버깅
- GUI 앱이지만 테스트, 로그, 로더 경로로 줄여서 추적해야 하는 문제
- 크래시, parse 실패, launch 실패, config/state mismatch
- timing, sample-rate, callback, threading, queue handoff 문제
- audio, render, input, file/path, cache 경계 디버깅
- CMake/MSBuild 기반 프로젝트의 좁은 repro + 빌드/테스트 검증

## What This Skill Is Not For

- 광범위한 아키텍처 재설계
- 단순 스타일 리팩터링
- 비-C++ 작업
- 근거 없이 로그만 늘리는 식의 탐색

## Workflow

이 스킬은 아래 순서를 강하게 유도합니다.

1. 재현 범위를 줄입니다.
2. 문제가 드러나는 경계를 찾습니다.
3. 필요한 지점에만 좁게 계측합니다.
4. 가능하면 테스트나 결정적 repro로 바꿉니다.
5. 진실을 소유한 레이어에서만 보수적으로 수정합니다.
6. 좁은 테스트부터 전체 빌드/검증까지 닫습니다.

핵심 철학은 `이론을 키우기 전에 버그를 작게 만든다` 입니다.

## Repository Structure

이 저장소는 Codex Agent Skills 구조를 따릅니다.

```text
cpp-sandbox-debug-skills/
├─ SKILL.md
├─ agents/
│  └─ openai.yaml
├─ LICENSE
└─ README.md
```

- `SKILL.md`: 실제 스킬 지침과 trigger description
- `agents/openai.yaml`: Codex app/skill UI용 메타데이터
- `README.md`: GitHub 공개 설명

## Installation

Codex 공식 Agent Skills 문서 기준으로, 저장소 안의 스킬은 `.agents/skills` 아래에 두면 인식됩니다.

예시:

```text
your-repo/
└─ .agents/
   └─ skills/
      └─ cpp-sandbox-debug/
         ├─ SKILL.md
         └─ agents/openai.yaml
```

이 저장소를 그대로 복사하거나 clone한 뒤, 스킬 폴더 이름을 `cpp-sandbox-debug`로 맞춰 `.agents/skills/` 아래에 두면 됩니다.

최근 Codex 버전에서는 다른 GitHub 저장소의 스킬을 installer로 가져오는 흐름도 지원됩니다. 다만 실제 설치 방식은 Codex 버전에 따라 조금 다를 수 있으니, 최신 Agent Skills 문서를 함께 확인하는 것이 안전합니다.

## Usage

명시적으로 쓰려면 프롬프트에서 스킬을 직접 호출하면 됩니다.

```text
Use $cpp-sandbox-debug to debug this C++ crash.
```

또는 Codex가 작업 설명과 `SKILL.md`의 description을 보고 암묵적으로 선택할 수 있습니다.

잘 맞는 요청 예시:

- "이 C++ 크래시를 샌드박스 환경에서 디버깅해줘"
- "GUI 디버거 없이 이 parse failure 원인을 좁혀줘"
- "MSBuild는 되는데 런타임 launch가 실패하는 이유를 찾아줘"
- "audio callback glitch를 테스트와 좁은 로그로 추적해줘"

## Why This Exists

C++ 디버깅은 도구가 제한되는 순간 금방 감에 의존하게 됩니다. 이 스킬은 그런 상황에서:

- 어디서부터 읽을지
- 어떤 경계를 먼저 볼지
- 어떤 로그를 남기고 어떤 로그를 피할지
- 테스트를 언제 붙일지
- 수정 범위를 어디까지로 제한할지

를 더 일관되게 만들기 위해 만들어졌습니다.

## Compatibility

이 스킬은 Codex의 Agent Skills 형식을 기준으로 작성되었습니다.

- `SKILL.md` 필수
- `agents/openai.yaml` 선택 메타데이터 포함
- instruction-only 스킬로 동작

## License

이 저장소의 라이선스는 루트의 [LICENSE](./LICENSE)를 따릅니다.
