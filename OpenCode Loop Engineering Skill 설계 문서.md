# OpenCode Loop Engineering Skill 설계

## 1. 개요

`loop-engineering`은 OpenCode와 같은 AI Coding Agent가 장시간 작업을 수행할 때 사용하는 범용 Skill이다.

단순히 하나의 프롬프트를 실행하는 것이 아니라 다음 사이클을 반복한다.

```text
Goal
 ↓
현재 상태 파악
 ↓
Planning
 ↓
Implementation
 ↓
Verification
 ↓
Progress 기록
 ↓
다음 작업 결정
 ↺
```

핵심 철학은 다음과 같다.

> **Conversation is disposable. State is persistent.**

대화 Context 자체를 장기 기억으로 사용하지 않는다.

프로젝트의 목표, 진행 상태, 중요한 의사결정을 외부 Markdown 파일에 지속적으로 기록하고, 새로운 세션에서도 해당 파일을 읽어 작업 상태를 복구한다.

---

# 2. 목적

이 Skill의 목적은 다음과 같다.

- 장시간 AI Coding Agent 작업 지원
- Context Window 한계 완화
- 세션 초기화 이후 작업 복구
- 자동 계획 수립
- 반복적인 구현
- 테스트 기반 검증
- 실패 원인 분석
- 진행 상태 기록
- Multi-Agent 작업 지원
- 무한 루프 방지
- Human-in-the-loop 지원

궁극적으로 다음과 같은 작업을 가능하게 한다.

```text
사용자

"이 프로젝트 완성해."
        ↓
Loop Engineering Skill
        ↓
목표 분석
        ↓
계획
        ↓
구현
        ↓
테스트
        ↓
검증
        ↓
상태 기록
        ↓
다음 작업
        ↓
...
        ↓
완료 조건 만족
```

---

# 3. 기본 구조

권장 Skill 구조는 다음과 같다.

```text
.skills/
└── loop-engineering/
    ├── SKILL.md
    │
    ├── templates/
    │   ├── GOAL.md
    │   ├── PROGRESS.md
    │   └── DECISIONS.md
    │
    └── scripts/
        └── check_state.py
```

프로젝트에서는 다음 상태 파일을 사용한다.

```text
project/
├── AGENTS.md
├── GOAL.md
├── PROGRESS.md
├── DECISIONS.md
├── src/
└── tests/
```

각 파일의 역할은 다음과 같다.

| 파일 | 역할 |
|---|---|
| `AGENTS.md` | 프로젝트 전체 Agent 규칙 |
| `GOAL.md` | 최종 목표와 완료 조건 |
| `PROGRESS.md` | 현재 작업 상태 |
| `DECISIONS.md` | 중요한 기술적 결정 및 이유 |

---

# 4. 핵심 구성 요소

Loop Engineering은 다음 다섯 가지 단계로 구성한다.

```text
Planning
   ↓
Implementation
   ↓
Verification
   ↓
Progress Tracking
   ↓
Context Recovery
   ↺
```

각 단계는 독립적인 책임을 갖는다.

---

# 5. Planning

Planning 단계에서는 현재 상태를 분석하고 다음에 수행할 작업을 결정한다.

Agent는 다음 정보를 확인한다.

```text
GOAL.md
PROGRESS.md
DECISIONS.md
AGENTS.md
Repository
Tests
```

그 후 현재 목표를 달성하기 위해 가장 중요한 다음 작업을 선택한다.

원칙은 다음과 같다.

```text
One iteration
=
One clearly defined objective
```

하나의 iteration에서 지나치게 많은 작업을 수행하지 않는다.

가능하면 검증 가능한 최소 단위로 작업을 나눈다.

---

# 6. Implementation

Planning 단계에서 선택한 작업을 구현한다.

예:

```text
Current Objective

Implement Impala batch query
```

Agent는 필요한 코드를 분석하고 최소 범위의 변경을 수행한다.

가능하면 기존 구조를 존중한다.

불필요한 리팩터링이나 관련 없는 변경은 피한다.

---

# 7. Verification

구현 이후 반드시 결과를 검증한다.

가능한 검증 방법:

```text
Unit Test
Integration Test
Lint
Type Check
Build
Runtime Test
Query Test
API Test
Manual Verification
```

Agent는 검증 없이 작업 완료를 선언해서는 안 된다.

예:

```text
Implementation
      ↓
pytest
      ↓
127 passed
2 failed
      ↓
Failure Analysis
      ↓
Fix
      ↓
pytest
```

---

# 8. 실패 처리

검증이 실패하면 즉시 새로운 기능 개발로 넘어가지 않는다.

다음 순서를 따른다.

```text
Failure
   ↓
원인 분석
   ↓
수정 가능?
   │
   ├── YES → Fix
   │          ↓
   │       Verify
   │
   └── NO → Blocker 기록
              ↓
          Human Escalation
```

동일한 실패를 무한 반복하지 않는다.

---

# 9. Progress Tracking

각 iteration이 끝날 때 `PROGRESS.md`를 업데이트한다.

예:

```markdown
# Project Progress

## Current Goal

ETL Pipeline 구현

## Completed

- DB schema
- connection pool
- query module

## Current

Impala batch query 구현

## Blockers

Impala timeout 발생

## Next

1. batch size 1000 테스트
2. timeout 측정
3. retry 정책 구현

## Verification

pytest

127 passed
2 failed

## Last Updated

2026-10-06
```

`PROGRESS.md`는 대화 내용을 그대로 저장하는 로그가 아니다.

현재 상태를 빠르게 복구할 수 있는 **압축된 프로젝트 상태**여야 한다.

---

# 10. Decision Tracking

중요한 기술적 판단은 `DECISIONS.md`에 기록한다.

예:

```markdown
# Decisions

## DEC-001

### Decision

Pandas 대신 Polars 사용

### Reason

대규모 ETL 처리 성능 개선

### Alternatives

- Pandas
- DuckDB

### Status

Accepted
```

이 파일을 사용하는 이유는 새로운 Agent가 과거의 결정을 모르고 동일한 논의를 반복하거나 이미 버린 설계로 되돌아가는 것을 방지하기 위함이다.

---

# 11. Context Recovery

Context Window가 커지거나 새로운 세션이 시작되면 기존 Conversation 전체를 복원하려 하지 않는다.

다음 파일을 이용해 상태를 복구한다.

```text
AGENTS.md
   +
GOAL.md
   +
PROGRESS.md
   +
DECISIONS.md
   +
Repository
   =
Current State
```

새로운 세션에서 Agent는 다음 순서로 동작한다.

```text
Read AGENTS.md

Read GOAL.md

Read PROGRESS.md

Read DECISIONS.md

Inspect Repository

Inspect Tests

Reconstruct State

Continue Loop
```

핵심 원칙:

> Conversation history는 작업 상태의 Source of Truth가 아니다.

프로젝트 파일이 Source of Truth가 되어야 한다.

---

# 12. GOAL.md

`GOAL.md`는 프로젝트의 최종 목적을 정의한다.

예:

```markdown
# Final Goal

사내 데이터 분석 시스템 완성

## Requirements

- Impala 데이터 조회
- ETL Pipeline
- MariaDB 저장
- Streamlit UI
- 사용자 인증

## Acceptance Criteria

- ETL 정상 동작
- 데이터 저장 정상
- Streamlit 실행 가능
- 주요 기능 테스트 통과
- Critical Error 없음
```

Acceptance Criteria는 Loop 종료 여부를 판단하는 핵심 기준이다.

---

# 13. 기본 Loop

Agent는 다음 루프를 반복한다.

```text
while goal_not_completed:

    read_goal()

    read_progress()

    inspect_repository()

    determine_next_task()

    implement()

    verify()

    if failure:
        analyze_failure()
        fix()

    update_progress()

    evaluate_stop_conditions()
```

실제 구현에서는 반드시 iteration 제한 및 종료 조건을 사용한다.

---

# 14. Stop Conditions

무한 루프를 방지하기 위해 명확한 Stop Condition을 정의한다.

다음 조건에서는 작업을 중단한다.

### Goal Completed

```text
모든 Acceptance Criteria 만족
```

### Repeated Failure

```text
동일한 오류가 3회 이상 반복
```

### Iteration Limit

예:

```text
max_iterations = 30
```

### Human Decision Required

예:

```text
Architecture 변경

Database schema 변경

API 계약 변경

보안 정책 변경
```

### Permission Required

예:

```text
관리자 권한

외부 시스템 인증

Production 접근
```

### Destructive Operation

예:

```text
DROP TABLE

rm -rf

Production 데이터 변경

Git history rewrite
```

이 경우 자동으로 진행하지 않고 사용자에게 판단을 요청한다.

---

# 15. Loop 상태

Loop는 명시적인 상태를 갖는 것이 좋다.

```text
PLANNING

IMPLEMENTING

VERIFYING

FIXING

BLOCKED

COMPLETED
```

예:

```text
PLANNING
   ↓
IMPLEMENTING
   ↓
VERIFYING
   ↓
 ┌─┴───────┐
PASS      FAIL
 ↓          ↓
RECORD    FIXING
 ↓          ↓
NEXT ←──────┘
```

---

# 16. Multi-Agent 확장

Loop Engineering Skill은 Multi-Agent 구조로 확장할 수 있다.

예:

```text
               Orchestrator
                    │
         ┌──────────┼──────────┐
         ↓          ↓          ↓
      Planner    Developer   Researcher
         │          │          │
         └──────────┼──────────┘
                    ↓
                  Tester
                    ↓
                 Reviewer
                    ↓
               PASS / FAIL
                    ↓
               PROGRESS.md
                    ↓
                Next Loop
```

---

# 17. Agent 역할

## Orchestrator

전체 Loop를 관리한다.

직접 구현하는 것보다 다음 작업 결정과 Agent 배분에 집중한다.

---

## Planner

현재 상태를 분석하고 다음 작업을 결정한다.

---

## Developer

실제 코드 변경을 담당한다.

---

## Researcher

문서, 코드베이스 또는 필요한 기술 정보를 조사한다.

---

## Tester

테스트와 실행 검증을 담당한다.

---

## Reviewer

구현 결과가 요구사항을 충족하는지 독립적으로 검토한다.

---

# 18. Maker / Checker 패턴

가능하면 구현 Agent와 검증 Agent를 분리한다.

```text
Developer
    ↓
Implementation
    ↓
Tester
    ↓
Reviewer
    ↓
PASS / FAIL
```

같은 Agent가 자신의 결과를 검증하는 것보다 독립적인 검증 역할을 두는 것을 권장한다.

즉:

```text
Maker ≠ Checker
```

---

# 19. Skill과 Agent의 역할

두 개념을 명확하게 구분한다.

### Skill

```text
HOW TO WORK
```

어떻게 작업할 것인지 정의한다.

### Agent

```text
WHO DOES THE WORK
```

누가 작업할 것인지 정의한다.

### State Files

```text
WHERE ARE WE NOW
```

현재 상태를 기록한다.

### Loop

```text
WHAT HAPPENS NEXT
```

다음 작업을 결정하고 다시 실행한다.

이를 합치면 다음 구조가 된다.

```text
             Skill
       "How to work"
              │
              ↓
         Orchestrator
              │
       ┌──────┼──────┐
       ↓      ↓      ↓
    Planner Developer Tester
       │      │      │
       └──────┼──────┘
              ↓
          State Files
              ↓
          Next Loop
              ↺
```

---

# 20. 권장 SKILL.md

초기 버전의 `SKILL.md`는 다음 구조로 작성할 수 있다.

```markdown
# Loop Engineering

## Purpose

Perform long-running development work using iterative
planning, implementation, verification and persistent state.

## Core Principle

Conversation is disposable.
State is persistent.

Never rely solely on conversation history for project state.

## On Start

1. Read AGENTS.md if available.
2. Read GOAL.md.
3. Read PROGRESS.md.
4. Read DECISIONS.md if available.
5. Inspect repository state.
6. Inspect relevant tests.
7. Reconstruct current project state.

## Loop

For every iteration:

1. Determine the current objective.
2. Select the highest-priority actionable task.
3. Keep the scope small and verifiable.
4. Implement the task.
5. Run appropriate verification.
6. Analyze failures.
7. Fix failures when reasonably possible.
8. Update PROGRESS.md.
9. Record important architectural decisions.
10. Evaluate stop conditions.
11. Continue with the next iteration when appropriate.

## Verification

Never claim completion without verification.

Use available:

- tests
- lint
- type checks
- builds
- runtime checks
- integration checks

## Progress Tracking

PROGRESS.md must represent the current project state,
not a verbose conversation log.

Always maintain:

- completed work
- current work
- blockers
- next actions
- latest verification results

## Context Recovery

When context becomes large or a new session begins:

1. Do not attempt to preserve the entire conversation.
2. Persist important state to project files.
3. Re-read state files.
4. Inspect repository state.
5. Continue from reconstructed state.

## Stop Conditions

Stop when:

- all acceptance criteria are satisfied
- the same failure occurs repeatedly
- iteration limit is reached
- human judgment is required
- credentials or permissions are required
- destructive actions require approval
- the next action cannot be safely determined

## Completion

Before declaring the project complete:

1. Read GOAL.md again.
2. Check every acceptance criterion.
3. Run final verification.
4. Update PROGRESS.md.
5. Summarize completed work.
6. Report remaining limitations.
```

---

# 21. 향후 확장

초기 버전 이후 다음 기능을 추가할 수 있다.

### Loop State Machine

```text
PLANNING
IMPLEMENTING
VERIFYING
FIXING
BLOCKED
COMPLETED
```

### 자동 상태 검사

```text
scripts/check_state.py
```

등을 이용해 다음 항목을 검사할 수 있다.

```text
GOAL.md 존재 여부

PROGRESS.md 존재 여부

Git working tree 상태

최근 commit

테스트 결과

현재 iteration

반복 오류 횟수
```

### Context Budget 관리

Context가 일정 수준 이상 증가하면:

```text
현재 상태 정리
    ↓
PROGRESS.md 업데이트
    ↓
중요 결정 DECISIONS.md 저장
    ↓
불필요한 Context 제거
    ↓
상태 복구
```

하도록 만들 수 있다.

### Git 연동

각 성공한 iteration마다 필요하면:

```text
Implementation
      ↓
Verification
      ↓
PASS
      ↓
PROGRESS update
      ↓
Git commit
```

패턴을 사용할 수 있다.

단, 자동 commit 여부는 프로젝트 정책에 따라 선택한다.

---

# 22. 최종 목표

이 Skill의 목표는 단순히 AI가 오랫동안 실행되도록 만드는 것이 아니다.

다음과 같은 **자율적인 개발 루프**를 만드는 것이다.

```text
              FINAL GOAL
                  │
                  ↓
           Understand State
                  ↓
               Plan
                  ↓
             Implement
                  ↓
              Verify
                  ↓
          ┌───────┴───────┐
        FAIL             PASS
          │                │
          ↓                ↓
     Analyze/Fix      Record State
          │                │
          └───────┬────────┘
                  ↓
            Goal reached?
             │          │
            NO         YES
             │          │
             ↺          ↓
                     COMPLETE
```

최종적으로 사용자는 다음과 같은 간단한 명령만으로 장시간 작업을 시작할 수 있어야 한다.

```text
Use the loop-engineering skill.

Read GOAL.md and continue working on the project
until the acceptance criteria are satisfied or
a stop condition is reached.
```

핵심 설계 철학은 다음 세 문장으로 요약한다.

> **Conversation is disposable.**

> **State is persistent.**

> **Every iteration must produce either verified progress or useful information about why progress is blocked.**