---
name: brand-to-vercel
description: 브랜드 인터뷰부터 4종 HTML 산출물 제작, Vercel 배포까지 전 과정을 한 번에 진행한다. "내 브랜드 만들어서 배포해줘", "가상 회사 사이트 만들어줘", "브랜드 키트 만들고 Vercel에 올려줘"처럼 브랜드 생성 + 배포를 함께 요청할 때 사용한다.
---

# 브랜드 키트 → Vercel 배포 스킬

인터뷰 → 디자인 시트 → 4종 HTML 산출물 → Vercel 배포까지 전 과정을 실행한다.

## 전체 흐름

```
Phase 1  브랜드 인터뷰       → 01_brand.md
Phase 2a 회사명·슬로건 선택  → (메인 대화에서 직접)
Phase 2b 디자인 시트 생성    → brand-skill/
Phase 3  4종 HTML 산출물     → outputs/
Phase 4  Vercel 배포         → 퍼블릭 URL
```

작업 폴더: `_workspace/brand-kit/<slug>/`

---

## Phase 1 — 브랜드 인터뷰

`brand-kit:brand-interview` 스킬을 호출한다.

끌어낼 7가지:
1. 씨앗(계기)
2. 하고 싶은 일
3. 어려움/악당
4. 잊고 있던 꿈
5. 말 거는 대상(청중)
6. 한 줄 소개
7. 무드(색·글씨 느낌·배경 톤)

산출물:
- `00_interview-transcript.md` — 대화 원본
- `01_brand.md` — 정리본 + 회사명 후보 5개 + 슬로건 후보

게이트 ①: 정리본 보여주고 "고칠 데 있으세요?" 확인 후 진행.

---

## Phase 2 — 디자인 시트

### 2a. 선택 (반드시 메인 대화에서 — 서브에이전트로 넘기지 말 것)

1. 회사명 후보 5개 + 슬로건 후보 보여주고 하나씩 고르게 한다.
2. 브랜드 결에 맞는 레퍼런스 사이트 2~3개를 이유 한 줄과 함께 추천한다.
3. 참가자가 하나를 고를 때까지 기다린다.
4. 슬러그 확정: 회사명을 소리 나는 대로 로마자로 (예: 이음→`ieum`).

### 2b. 디자인 시트 생성

`brand-kit:brand-sheet` 스킬 호출.

산출물 (`brand-skill/`):
- `SKILL.md` — 브랜드 스킬 정의
- `color-typography-specs.md` — hex·폰트·복붙 CSS
- `usage-examples.md` — 형태별 예시
- `mark.svg` — 워드마크 SVG

**폰트 규칙:**
- 로고 전용: `Black Han Sans` → `--logo-font`
- 제목(h1/h2): `Pretendard 800` → `--display` (Black Han Sans 절대 금지)
- 본문: `Pretendard 400` → `--sans`

**CSS 토큰:**
```css
:root {
  --primary: #FF6B35;  --primary-2: #FF9A6C;
  --bg: #FFFAF7;       --surface: #FFFFFF;
  --ink: #1C1C2E;      --ink2: #6B7280;
  --accent: #2563EB;   --line: #E8E4E0;
  --logo-font: 'Black Han Sans', sans-serif;
  --display: 'Pretendard', 'Apple SD Gothic Neo', sans-serif;
  --sans:    'Pretendard', 'Apple SD Gothic Neo', sans-serif;
}
```

포인트색(`--accent`)은 화면당 한 곳만.

게이트 ②: 색·폰트·로고 보여주고 "이 느낌 맞아요?" 확인.

---

## Phase 3 — 산출물 4종

저장 위치: `_workspace/brand-kit/<slug>/outputs/`

### 공통 규칙
- `<head>`에 폰트 CDN 2개:
  ```html
  <link href="https://fonts.googleapis.com/css2?family=Black+Han+Sans&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css">
  ```
- `:root` CSS 토큰 인라인
- 외부 이미지 0개 (SVG/CSS만)
- 포인트색(`--accent`) 화면당 1곳
- 모든 페이지에 **"← 홈으로" 버튼** (`index.html`로 이동)
- 모바일 반응형: 브레이크포인트 960px + 640px

### index.html — 허브 페이지
4종 페이지로 이동하는 카드 그리드. 모바일에선 1열로.

### web.html — 랜딩 페이지
섹션: 헤더(sticky+햄버거) → 히어로(H1+CTA+우측 비주얼) → 통계 바(어두운 배경) → 문제(2×2 카드) → 피처(3카드) → 스토리(인용) → CTA 푸터(그라데이션)

IntersectionObserver 스크롤 fade-up 애니메이션 적용.

`.logo` 클래스는 `font-family: var(--logo-font)` 사용.
`h1`, `h2`는 `font-family: var(--display); font-weight: 800` 사용.

### ppt.html — 슬라이드 6장
- 1280×720 고정, `transform:scale(Math.min(w/1280, h/720))` auto-fit
- 키보드 ←/→ + `location.hash #1~#6` 점프
- 터치 스와이프 지원
- 컨트롤 바: `<a href="index.html">🏠 홈</a>` 버튼 포함
- 슬라이드 전환: opacity fade (.3s)

슬라이드 구성:
1. 표지 — 좌우 분할
2. 문제 — 2×2 카드
3. 전환 — 어두운 배경 임팩트 문구
4. 과정 — 3카드 + 숫자 뱃지
5. 숫자 — 통계 3개
6. 맺음 — 연락처 패널

### cardnews.html — 정사각 카드 5장
- 가로 필름스트립 (`scroll-snap-type:x mandatory`)
- 모바일: 세로 스크롤 전환 (`@media(max-width:640px)`)
- 카드 내 워드마크 워터마크: `font-family: var(--logo-font)`
- `page-header .logo`: `font-family: var(--logo-font)`
- 카드 순서: 표지(주색) → 문제 → 전환(어두운) → 방법 → CTA(주색)
- 모든 h2: `font-weight: 800`

### namecard.html — 명함 + 메일서명
- 명함 앞면: 주색 배경, 로고 중앙 (`font-family: var(--logo-font)`)
- 명함 뒷면: 밝은 배경, 이름·연락처
- 메일 서명: 로고(`var(--logo-font)`) | 구분선 | 이름·직함·연락처·슬로건
- `.logo`, `.logo-small`, `.sig-logo`: 모두 `font-family: var(--logo-font)`
- `.name`, `.sig-name`: `font-weight: 700`

---

## Phase 4 — Vercel 배포

```powershell
# Vercel CLI 없으면 먼저 설치
npm install -g vercel

# outputs/ 폴더에서 배포
cd _workspace/brand-kit/<slug>/outputs
vercel --prod --yes
```

Bash가 아닌 **PowerShell**로 실행해야 한다. (Bash에서는 vercel 명령을 못 찾는 경우가 있음)

배포 후 퍼블릭 URL 출력. 이후 업데이트도 동일 명령어.

---

## 게이트 요약

| 단계 | 확인 내용 |
|------|-----------|
| ① 인터뷰 후 | `01_brand.md` 검토 → OK |
| ② 디자인 시트 후 | 색·폰트·로고 확인 → OK |
| ③ 산출물 후 | 4종 미리보기 → 부분 수정 |
| ④ 배포 후 | 퍼블릭 URL 전달 |

---

## 배포 전 체크리스트

- [ ] CSS 토큰이 `color-typography-specs.md`와 동일한가
- [ ] 제목 h1/h2: `Pretendard 800` (Black Han Sans는 로고·워터마크만)
- [ ] 포인트색이 화면당 1곳인가
- [ ] 모든 페이지에 "← 홈으로" 버튼이 있는가
- [ ] 모바일에서 레이아웃이 깨지지 않는가
- [ ] `ppt.html` hash #1~#6 점프가 작동하는가
- [ ] 외부 이미지 0개 (폰트 CDN만 허용)
- [ ] Vercel 배포: PowerShell에서 `vercel --prod --yes`
