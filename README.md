# opencode-skills

OpenCode용 스킬 모음이다. 각 스킬은 `skills/<skill-name>/`에 독립적으로 관리한다.

| 스킬 | 기능 |
| --- | --- |
| [do-loop](skills/do-loop/SKILL.md) | 계획·구현·검증을 반복하고 Markdown 상태 파일로 작업 복구 |

`do-loop`는 제공된 `OpenCode Loop Engineering Skill 설계 문서.md`를 바탕으로 만들었다.

## 구성

```text
skills/do-loop/
├── SKILL.md
├── templates/
│   ├── GOAL.md
│   ├── PROGRESS.md
│   └── DECISIONS.md
└── references/
    └── multi-agent.md
```

- `GOAL.md`: 목표, 범위, 관찰 가능한 완료 조건
- `PROGRESS.md`: 현재 작업, 검증 근거, 반복·실패 횟수, 재개 조건
- `DECISIONS.md`: 중요한 선택, 이유, 대안과 변경 이력

상태 파일은 스킬 설치 디렉터리가 아닌 **작업 대상 프로젝트 루트**에 생성된다.
기존 파일이 있으면 내용을 보존하며 필요한 정보만 보완한다.

## do-loop 설치

OpenCode가 설치되어 있어야 한다. 스킬 자체는 Markdown으로 구성되어 추가 런타임이나 패키지가 필요 없다.
아래 명령은 macOS/Linux 셸 기준이다. 비공개 저장소를 받으려면 GitHub 계정에 접근 권한이 있어야 한다.

원하는 상위 디렉터리에서 GitHub CLI로 복제한다.

```bash
gh repo clone jerry-leem/opencode-skills
cd opencode-skills
```

이 저장소 루트에서 전역 설치한다. 기존 `do-loop`가 있으면 덮어쓰지 않고 중단한다.

```bash
skill_root="${XDG_CONFIG_HOME:-$HOME/.config}/opencode/skills"
if [ -e "$skill_root/do-loop" ] || [ -L "$skill_root/do-loop" ]; then
  echo "이미 설치되어 있습니다: $skill_root/do-loop"
else
  mkdir -p "$skill_root" && cp -R skills/do-loop "$skill_root/do-loop"
fi
```

특정 프로젝트에만 설치하려면 저장소 루트에서 대상 프로젝트의 실제 절대 경로를 지정하고 실행한다.

```bash
project_root="/absolute/path/to/your-project"
if [ ! -d "$project_root" ]; then
  echo "프로젝트 경로를 확인하세요: $project_root"
elif [ -e "$project_root/.opencode/skills/do-loop" ] || [ -L "$project_root/.opencode/skills/do-loop" ]; then
  echo "이 프로젝트에 이미 do-loop가 설치되어 있습니다."
else
  mkdir -p "$project_root/.opencode/skills" && cp -R skills/do-loop "$project_root/.opencode/skills/do-loop"
fi
```

성공하면 설치 경로에 `SKILL.md`, `templates/`, `references/`가 생긴다.
프로젝트 로컬 설치 후에는 해당 프로젝트 디렉터리에서 OpenCode를 시작한다.
이 저장소의 `skills/`는 배포용 경로이며 복제만으로 자동 등록되지 않는다.
발견 경로와 필수 frontmatter는 [OpenCode 공식 스킬 문서](https://opencode.ai/docs/skills/)를 따른다.

## 사용

작업 대상 프로젝트에서 OpenCode에 다음처럼 요청한다.

```text
do-loop 스킬을 사용해줘.
GOAL.md를 읽고 완료 조건을 모두 검증하거나 중단 조건에 도달할 때까지 진행해줘.
```

목표 파일이 없다면 구체적인 목표를 함께 전달한다.

```text
do-loop 스킬을 사용해서 CSV 업로드 기능을 구현해줘.
완료 조건은 정상 파일 업로드, 잘못된 헤더 오류 표시, 관련 테스트 통과야.
최대 10회 반복하고 배포는 하지 마.
```

새 세션에서는 다음처럼 이어간다.

```text
do-loop 스킬로 AGENTS.md, GOAL.md, PROGRESS.md, DECISIONS.md를 읽고
실제 저장소 상태와 대조한 다음 중단된 작업을 이어가줘.
```

스킬은 에이전트가 `skill({ name: "do-loop" })`로 읽는 지침이다.
`/do-loop` 슬래시 명령이나 백그라운드 실행기는 포함하지 않는다.
세션 종료 후에는 사용자가 다시 실행해야 한다.

## 반복과 중단

기본 상한은 30회이며 동일 실패가 연속 3회 발생하면 중단한다.
실패 후 수정과 재검증도 다음 반복으로 계산한다.
새 세션에서도 반복·실패 기록을 유지한다.
상한 도달, 필요한 입력·권한 부재, 승인 범위 밖의 결정, 사용자 중단도 `BLOCKED`로 기록한다.
상한 확대나 반복 실패 후 새 시도는 사용자 지시와 변경 이유를 남긴다.

완료 조건이 모두 실제 검증된 경우에만 `COMPLETED`로 종료한다.
커밋·푸시·배포·데이터 삭제 등의 권한은 스킬을 호출했다고 자동 부여되지 않는다.
여러 에이전트는 사용 가능한 도구와 권한이 있을 때만 선택적으로 활용한다.

## 검증과 기여

빌드할 실행 코드와 패키지 의존성은 없다. Markdown 변경 시 frontmatter, 상대 링크,
설치 명령, 상태 전이·복구·중단 규칙을 점검한다.
작업 대상 프로젝트에서 다음 명령을 실행하면 설치된 스킬 목록에 `do-loop`가 나타나야 한다.

```bash
opencode debug skill
```

확인한 환경과 시나리오는 [검증 기록](docs/validation.md)에 구분해 기록한다.
자동 상태 검사 스크립트와 세션 재실행 플러그인은 포함하지 않는다.
