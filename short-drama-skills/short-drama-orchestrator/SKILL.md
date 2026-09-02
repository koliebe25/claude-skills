---
name: short-drama-orchestrator
description: "세로 9:16 숏폼 드라마 기획·제작 에이전트 팀을 조율하는 오케스트레이터. 드라마 기획부터 대본·이미지 프롬프트·영상 프롬프트·검수까지 전체 파이프라인을 실행. '숏폼 드라마 만들어줘', '드라마 기획/제작', '쇼츠/틱톡/릴스 드라마', 소재로 드라마 제작 요청 시 사용. 후속 작업: 드라마 결과 수정, 특정 회차/캐릭터/컷만 다시, 부분 재실행, 업데이트, 보완, 다시 실행, 이전 결과 개선, 대본/비주얼/영상만 다시 요청 시에도 반드시 이 스킬을 사용."
---

# Short Drama Orchestrator

세로 9:16 숏폼 드라마(틱톡/쇼츠/릴스, 회차 30초 내외) 기획·제작 에이전트 팀을 조율하여 **기획 문서 + 회차 대본 + 이미지 프롬프트 + 영상 프롬프트 + 검수 리포트**를 생성하는 통합 스킬.

## 실행 모드: 에이전트 팀

## 에이전트 구성

| 팀원 | 에이전트 타입 | 역할 | 스킬 | 출력 |
|------|-------------|------|------|------|
| story-architect | general-purpose | 기획·캐릭터 바이블 | drama-planning | `_workspace/01_story-architect_planning.md` |
| scriptwriter | general-purpose | 회차 대본 | screenwriting | `_workspace/02_scriptwriter_script.md` |
| visual-director | general-purpose | 컷/이미지 프롬프트 | visual-prompt | `_workspace/03_visual-director_*.md` |
| video-producer | general-purpose | 샷/영상 프롬프트 | video-prompt | `_workspace/04_video-producer_*.md` |
| qa-reviewer | general-purpose | 경계면 정합성 검수 | drama-qa | `_workspace/05_qa-reviewer_report.md` |

> 모든 팀원은 `.claude/agents/{name}.md` 정의를 따르고 `model: "opus"`로 호출한다. 각 Agent/TeamCreate 호출에 `model: "opus"`를 명시한다.

## 워크플로우

### Phase 0: 컨텍스트 확인 (후속 작업 지원)
1. `_workspace/` 존재 여부 확인
2. 실행 모드 결정:
   - **미존재** → 초기 실행, Phase 1로
   - **존재 + 부분 수정 요청**(예: "3화 대본만 다시", "주인공 외형 바꿔") → **부분 재실행**. 해당 에이전트만 재호출, 기존 산출물 중 대상만 덮어씀. 상류 변경(기획·캐릭터)이면 하류(대본/비주얼/영상)도 영향 범위 재실행
   - **존재 + 새 소재 제공** → **새 실행**. 기존 `_workspace/`를 `_workspace_{YYYYMMDD_HHMMSS}/`로 이동 후 Phase 1
3. 부분 재실행 시 이전 산출물 경로를 에이전트 프롬프트에 포함해 기존 결과를 읽고 피드백을 반영하도록 지시

### Phase 1: 준비
1. 사용자 입력 분석 — 소재/장르/톤, 회차 수, 플랫폼(미지정 시 기본값: 세로 9:16, 틱톡·쇼츠·릴스, 회차 30초 내외)
2. `_workspace/` 생성 (초기 실행 시)
3. 입력 자료를 `_workspace/00_input/`에 저장

### Phase 2: 팀 구성
```
TeamCreate(
  team_name: "short-drama-team",
  members: [
    { name: "story-architect", agent_type: "general-purpose", model: "opus",
      prompt: ".claude/agents/story-architect.md 역할 수행. drama-planning 스킬 사용. 소재: {입력}" },
    { name: "scriptwriter", agent_type: "general-purpose", model: "opus",
      prompt: ".claude/agents/scriptwriter.md 역할 수행. screenwriting 스킬 사용." },
    { name: "visual-director", agent_type: "general-purpose", model: "opus",
      prompt: ".claude/agents/visual-director.md 역할 수행. visual-prompt 스킬 사용." },
    { name: "video-producer", agent_type: "general-purpose", model: "opus",
      prompt: ".claude/agents/video-producer.md 역할 수행. video-prompt 스킬 사용." },
    { name: "qa-reviewer", agent_type: "general-purpose", model: "opus",
      prompt: ".claude/agents/qa-reviewer.md 역할 수행. drama-qa 스킬 사용." }
  ]
)
```
```
TaskCreate(tasks: [
  { title: "기획·캐릭터 바이블 작성", assignee: "story-architect" },
  { title: "회차 대본 집필", assignee: "scriptwriter", depends_on: ["기획·캐릭터 바이블 작성"] },
  { title: "기획↔대본 검수", assignee: "qa-reviewer", depends_on: ["회차 대본 집필"] },
  { title: "캐릭터 시트·컷 이미지 프롬프트", assignee: "visual-director", depends_on: ["회차 대본 집필"] },
  { title: "대본↔비주얼 검수", assignee: "qa-reviewer", depends_on: ["캐릭터 시트·컷 이미지 프롬프트"] },
  { title: "샷 리스트·영상 프롬프트", assignee: "video-producer", depends_on: ["캐릭터 시트·컷 이미지 프롬프트"] },
  { title: "비주얼↔영상·플랫폼 규격 최종 검수", assignee: "qa-reviewer", depends_on: ["샷 리스트·영상 프롬프트"] }
])
```

### Phase 3: 파이프라인 실행 (점진적 검수 포함)
**실행 방식:** 의존성 기반 파이프라인. 팀원이 공유 작업 목록에서 작업을 claim하고 수행.

**통신 규칙:**
- story-architect → scriptwriter/visual-director: 캐릭터 바이블·톤 SendMessage
- scriptwriter → visual-director: 씬별 강조 비주얼 SendMessage
- visual-director → video-producer: 컷 프롬프트·외형 앵커 SendMessage
- **qa-reviewer는 각 단계 완료 직후 직전 단계와 교차 비교(점진적 검수)** 후, FIX/BLOCK을 담당 에이전트에게 직접 SendMessage. 수정 후 해당 경계면만 재검증
- 모든 팀원은 완료 시 파일 저장 + TaskUpdate + 리더 알림

**리더 모니터링:** 유휴 알림 수신, 막힌 팀원에 SendMessage로 개입, TaskGet으로 진행률 확인.

### Phase 4: 통합
1. 모든 작업 완료 대기 (TaskGet)
2. 산출물 Read: 01~05
3. qa-reviewer 리포트의 미해결 FIX/BLOCK 확인 → 잔여 사안 정리
4. 최종 패키지 생성: `드라마_{제목}/` 디렉토리에 기획·대본·이미지프롬프트·영상프롬프트·검수리포트를 정리해 출력

### Phase 5: 정리
1. 팀원 종료 요청 (SendMessage)
2. 팀 정리 (TeamDelete)
3. `_workspace/` 보존 (감사 추적용)
4. 사용자에게 결과 요약 + 피드백 요청("개선할 부분/바꾸고 싶은 회차가 있나요?")

## 데이터 흐름
```
입력 → [story-architect] 01_planning
            │ (캐릭터 바이블)
            ↓
       [scriptwriter] 02_script ──QA(기획↔대본)
            │
            ↓
       [visual-director] 03_sheets+prompts ──QA(대본↔비주얼)
            │ (컷·외형 앵커)
            ↓
       [video-producer] 04_shotlist+video-prompts ──QA(비주얼↔영상·규격)
            │
            ↓
       [리더: 통합] → 드라마_{제목}/ 최종 패키지
```

## 에러 핸들링
| 상황 | 전략 |
|------|------|
| 팀원 1명 실패/중지 | 리더 감지 → SendMessage 상태 확인 → 재시작 또는 대체 팀원 생성 |
| 상류(기획) BLOCK | 하류 작업 보류, story-architect 수정 후 영향 범위만 재실행 |
| 캐릭터 외형 드리프트 | qa-reviewer가 탐지 → visual-director/video-producer에 앵커 일괄 반영 지시 |
| 회차 길이 2분 초과 | video-producer가 샷 압축 또는 scriptwriter/story-architect에 회차 분할 제안 |
| 동일 FIX 2회 반복 | 구조적 원인으로 보고 → 스킬/기획 차원 수정 제안 (하네스 진화) |
| 팀원 과반 실패 | 사용자에게 알리고 진행 여부 확인, 부분 결과 보존 |

## 테스트 시나리오

### 정상 흐름
1. 사용자가 소재 제공("재벌가 비밀 계약 결혼, 10부작 쇼츠")
2. Phase 1에서 플랫폼·회차 수 확정(세로 9:16, 10화, 회차 30초 내외)
3. Phase 2에서 5인 팀 + 7개 작업(의존성) 등록
4. Phase 3에서 기획→대본→(검수)→비주얼→(검수)→영상→(최종 검수) 순차 수행, qa가 점진 검수
5. Phase 4에서 `드라마_{제목}/`에 5종 산출물 통합
6. Phase 5에서 팀 정리 + 피드백 요청

### 에러 흐름
1. Phase 3에서 qa-reviewer가 "3화 컷에 주인공 머리색이 기획과 다름"(드리프트) BLOCK 판정
2. visual-director에게 SendMessage로 외형 앵커 일괄 수정 지시
3. visual-director가 3화 관련 컷 프롬프트 일괄 수정 → video-producer에 영향 샷 반영 요청
4. qa-reviewer가 비주얼↔영상 경계면만 재검증 → PASS
5. 최종 리포트에 "3화 캐릭터 드리프트 수정 완료" 기록 후 Phase 4 진행
