# claude-skills

Claude Code에서 직접 만들어 쓰는 스킬(Agent Skill) 백업 저장소.

## 구성

### `user-skills/` — 전역 스킬 (`~/.claude/skills/`)

| 스킬 | 하는 일 |
|---|---|
| `brand-to-vercel` | 브랜드 인터뷰 → 디자인 시트 → HTML 산출물 4종 → Vercel 배포까지 한 번에 |
| `harness` | 도메인별 에이전트 팀과 그 에이전트가 쓸 스킬을 함께 설계하는 메타 스킬 |
| `outfit-decompose` | 착장 사진 1장을 아이템별 제품컷으로 분해해 룩북 포스터를 만드는 웹앱 생성 |
| `science-simulation-hub` | 한국 중·고등 교육과정용 Canvas 인터랙티브 과학 시뮬레이션 + 학습 허브 |

### `short-drama-skills/` — 숏폼 드라마 제작 파이프라인

세로 9:16 / 회차 30초 내외 숏폼 드라마를 기획부터 검수까지 만드는 스킬 묶음.

| 스킬 | 하는 일 |
|---|---|
| `short-drama-orchestrator` | 아래 스킬들을 순서대로 조율하는 오케스트레이터 (진입점) |
| `drama-planning` | 컨셉·로그라인·훅·시즌 구조·캐릭터 바이블 |
| `screenwriting` | 회차별 대본, 씬 분할, 훅과 클리프행어 |
| `visual-prompt` | 캐릭터 레퍼런스 시트, 컷 분해, 이미지 생성 프롬프트 |
| `video-prompt` | 샷 리스트, 타임라인, image-to-video 프롬프트, 사운드/자막 큐 |
| `photoreal` | AI 티 안 나는 실사 인물 사진 프롬프트 설계 |
| `drama-qa` | 기획↔대본↔비주얼↔영상 경계면 정합성 검수 |

## 다른 PC에서 쓰기

```bash
git clone https://github.com/koliebe25/claude-skills.git
```

전역 스킬로 설치 (모든 프로젝트에서 사용):

```bash
cp -r claude-skills/user-skills/* ~/.claude/skills/
```

프로젝트 전용으로 설치 (해당 프로젝트에서만 사용):

```bash
mkdir -p .claude/skills && cp -r claude-skills/short-drama-skills/* .claude/skills/
```

Windows PowerShell:

```powershell
Copy-Item -Recurse claude-skills\user-skills\* $HOME\.claude\skills\
```

설치 후 Claude Code를 재시작하면 스킬이 인식된다.

## 백업 갱신

로컬에서 스킬을 고친 뒤:

```bash
cp -r ~/.claude/skills/brand-to-vercel ~/.claude/skills/harness ~/.claude/skills/outfit-decompose ~/.claude/skills/science-simulation-hub ~/claude-skills/user-skills/
cd ~/claude-skills && git add -A && git commit -m "update skills" && git push
```

## 여기 없는 것

- `computer-use`, `find-skills`, `orca-cli`, `orchestration` — Orca가 제공하는 스킬(심볼릭 링크)
- `vercel`, `brand-kit`, `humanize-korean` — 플러그인 마켓플레이스에서 설치한 것들
  (`anthropics/claude-plugins-official`, `yoonjechoi/hello-ryan-02`, `epoko77-ai/im-not-ai`)
- claude.ai 계정에 업로드한 스킬 — 로컬 파일이 아니라서 여기 담기지 않는다.
  claude.ai 설정에서 따로 내려받아야 한다.
