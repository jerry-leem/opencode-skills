# do-loop 검증 기록

검증일: 2026-10-06. 환경: macOS, OpenCode 1.18.30.

## 실행한 검증

| 항목 | 방법 | 결과 |
| --- | --- | --- |
| 스킬 형식 | skill-creator의 `quick_validate.py skills/do-loop` | `Skill is valid!` |
| 프로젝트 설치 | 임시 Git 프로젝트의 `.opencode/skills/do-loop/`에 스킬 전체 복사 | 설치 완료 |
| OpenCode 프로젝트 검색 | 임시 프로젝트에서 `opencode --pure debug skill` | 이름, 설명, 프로젝트 설치 경로 확인 |
| 전역 설치 | 임시 XDG 설정 디렉터리의 `opencode/skills/do-loop/`에 전체 복사 | 원본과 `diff -rq` 일치 |
| OpenCode 전역 검색 | 임시 `XDG_CONFIG_HOME`을 지정하고 `opencode --pure debug skill` | 전역 경로의 `do-loop` 확인 |
| 기존 설치 | 설치 경로 존재 조건 재실행 | 덮어쓰기 없이 중단하는 분기 확인 |
| 잘못된 프로젝트 경로 | 존재하지 않는 경로로 설치 조건 실행 | 복사 전 거부 분기 확인 |

검증기는 개발 환경의 Codex skill-creator에 있는 도구이며 이 저장소의 런타임 의존성이 아니다.
시스템 Python에 PyYAML이 없어 `uv run --with pyyaml python`으로 격리 실행했다.
OpenCode 전용 선택 필드 `compatibility`는 해당 범용 검증기가 지원하지 않아,
동일 정보를 양쪽에서 허용하는 `metadata.platform`으로 기록했다.

스킬 목록 출력이 큰 환경에서는 파이프 출력이 잘릴 수 있어 파일로 저장한 후 확인했다.
OpenCode 1.18.30에서 검증용 임시 프로젝트를 실행 디렉터리로 사용한 명령은 다음과 같다.

```bash
opencode --pure debug skill > /tmp/do-loop-skills.json
jq '.[] | select(.name == "do-loop") | {name, location, description}' /tmp/do-loop-skills.json
```

위 JSON 파일은 확인용 출력이며 저장소에 포함하지 않는다.

## 지침을 대조한 시나리오

아래는 문서의 상태 전이와 예외 규칙을 수동으로 대조한 결과다.
모델이 실제로 작업을 수행한 E2E 결과를 의미하지 않는다.

| 입력 상태 | 지침상 다음 행동 | 점검 |
| --- | --- | --- |
| 상태 파일이 없는 새 목표 | 없는 파일만 생성하고 목표·범위·검증 방법 구체화 | 일치 |
| `Iteration: 7`, 구현 도중 세션 종료 | 실제 변경 조사 후 7회차의 검증·기록 마무리 | 일치 |
| 같은 오류가 3회 발생 | `BLOCKED`, 실패 근거와 재개 조건 기록 | 일치 |
| 관련 없는 테스트만 성공 | 남아 있는 동일 실패 횟수 보존 | 일치 |
| 30회 종료, 완료 조건 미충족 | `iteration_limit`로 중단, 자동 상한 초기화 금지 | 일치 |
| 30회 종료, 완료 조건 전체 검증 | 완료 판정 우선, `COMPLETED`로 종료 | 일치 |
| 기존 파일 형식이 템플릿과 다름 | 기존 내용과 동등한 필드 재사용 | 일치 |
| 커밋·푸시가 이미 승인됨 | 재승인 없이 범위 내 실행 | 일치 |
| 미승인 파괴 작업 필요 | 실행 전 판단 요청 | 일치 |
| 하위 에이전트 도구 없음 | 단일 에이전트로 순차 수행 | 일치 |
| 검증 도구나 인증 부재 | 성공 처리하지 않고 제한과 재개 조건 기록 | 일치 |

## 검증 범위와 한계

Markdown 파일 7개의 상대 링크 6개가 모두 실제 파일을 가리키는 것을 확인했다.
`git diff --cached --check`에서 공백 오류가 없었다.
스킬과 템플릿은 Markdown이므로 빌드·타입 검사·실행 코드 단위 테스트는 해당하지 않는다.
`.gitignore`에는 이 환경의 LSP 서버가 설정되어 있지 않아 LSP 진단은 수행되지 않았다.
모델을 호출해 장시간 구현·세션 복구·다중 에이전트를 끝까지 실행하는 E2E 검증은 수행하지 않았다.
실제 반복 수행의 품질은 선택한 모델, 프로젝트 검증 수단, OpenCode 권한 설정에도 영향을 받는다.

설치 규칙은 [OpenCode 공식 스킬 문서](https://opencode.ai/docs/skills/)를 확인했다.
검증은 임시 디렉터리에서 수행했으며 사용자 전역 스킬 설치는 변경하지 않았다.

## Codex 지원 및 전역 설치 검증

추가 검증일: 2026-10-06. 환경: macOS, Codex CLI 0.160.0, OpenCode 1.18.30.
이번에는 사용자 요청에 따라 `~/.codex/skills/do-loop/`에 실제로 전역 설치했다.

| 항목 | 방법 | 결과 |
| --- | --- | --- |
| 원본·설치본 스킬 형식 | 양쪽 경로에 `quick_validate.py` 실행 | 모두 `Skill is valid!` |
| 설치 파일 일치 | 원본과 설치본을 `diff -rq`로 비교 | 차이 없음 |
| Codex 메타데이터 | PyYAML 파싱, 설명 길이, `$do-loop` 기본 프롬프트, 자동 선택 정책 검사 | 통과 |
| 실제 스킬 발견 | `codex debug prompt-input`의 모델 입력 목록 확인 | 사용자 전역 `do-loop/SKILL.md` 등록 확인 |
| OpenCode 호환성 | 갱신된 폴더를 임시 프로젝트에 설치 후 `opencode --pure debug skill` 실행 | 이름·프로젝트 설치 경로 확인 |
| 현재 대화 적용 준비 | 전역 설치본의 `SKILL.md` 직접 읽기 | 지침 로드 완료 |

Codex의 모델 입력 전체에는 개인 환경 정보가 포함될 수 있어 저장소에 보관하지 않았다.
확인 명령은 다음과 같다. 모델 추론을 호출하지 않는 검색·등록 검증이다.

```bash
codex debug prompt-input 'do-loop 스킬 검색 확인' > /tmp/do-loop-codex-input.json
rg -o 'do-loop[^\\]*' /tmp/do-loop-codex-input.json
```

YAML 언어 서버는 설치되어 있지 않아 PyYAML 파싱과 실제 Codex 검색으로 검증했다.
장시간 반복 개발과 다중 세션 복구의 모델 E2E 검증은 수행하지 않았다.
호출·발견·메타데이터 규칙은 [공식 OpenAI 문서](https://learn.chatgpt.com/docs/build-skills)를 확인했다.
