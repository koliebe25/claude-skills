# 허브 페이지 + 마우스 따라오는 로봇 배경

## 1. 허브 `index.html` 구조

허브는 시뮬레이션 카드를 카테고리별로 묶은 단일 HTML이다. 시뮬레이션과 달리 폰트는 `./fonts/`로
참조한다(허브가 루트에 있으므로).

### head / 전역 스타일 핵심

```css
:root{ --bg:#070a12; --text:#e6ecff; --dim:#9fb0d8; --muted:#6f7aa0;
  --card:rgba(22,28,46,0.72); --border:rgba(150,170,220,0.16); }
body{
  background:
    radial-gradient(120% 80% at 50% -10%, #14305a 0%, #0a1730 40%, #070a12 75%) fixed,
    #070a12;
  color:var(--text);
  font-family:'Pretendard',-apple-system,"Segoe UI","Malgun Gothic",system-ui,sans-serif;
  min-height:100vh; padding:48px 22px 70px;
}
.wrap{ max-width:1080px; margin:0 auto; }
.grid{ display:grid; grid-template-columns:repeat(auto-fill, minmax(250px,1fr)); gap:14px; }
```

### 헤더 + 범례

```html
<header>
  <h1>🔬 과학 시뮬레이션 학습 허브</h1>
  <p>교과서 속 원리를 직접 만지며 배우는 인터랙티브 실험실.<br>아래 카드를 누르면 시뮬레이션이 열립니다.</p>
  <span class="count">전체 N종 · 마우스/터치로 직접 조작 · 각 화면의 '📖 이론 배우기'로 설명 보기</span>
  <div class="legend">
    <span><i class="dot basic"></i>기초 · 중학교</span>
    <span><i class="dot adv"></i>심화 · 고등학교</span>
    <span><i class="dot fusion"></i>융합 · 교과 밖</span>
  </div>
  <p class="note">※ 2015·2022 개정 교육과정 기준의 대략적 위치·단원이에요. 교과서·출판사에 따라 다룰 수 있어요.</p>
</header>
```

### 카테고리 섹션 + 카드

개념을 의미 단위로 묶는다. 실제 사용된 분류 예: **물리 / 화학 / 자연·창발 / 생물 / 천체·지구·카오스**.
각 섹션은 색점(`sdot`) + 제목 + 작은 태그를 가진다.

```html
<section>
  <div class="sec-head"><span class="sdot" style="background:#378add"></span><h2>물리</h2><span class="tag">역학 · 파동 · 전기 · 빛</span></div>
  <div class="grid">
    <a class="card" href="./projectile-motion-simulation/index.html">
      <span class="accent" style="background:#378add"></span>
      <span class="go">→</span>
      <span class="emoji">🎯</span>
      <h3>포물선 운동</h3>
      <p>발사 각도·속력을 바꾸며 속도 벡터 분해와 v-t 그래프를 본다.</p>
      <div class="badges">
        <span class="badge basic">중3 기초 『운동과 에너지』</span>
        <span class="badge adv">물리학Ⅱ 『평면 운동』</span>
      </div>
    </a>
    <!-- ... 카드 반복 ... -->
  </div>
</section>
```

카드 스타일 핵심:

```css
a.card{ display:block; text-decoration:none; color:inherit; background:var(--card);
  border:0.5px solid var(--border); border-radius:16px; padding:18px 18px 16px;
  position:relative; overflow:hidden; transition:transform .14s, border-color .14s, background .14s; }
a.card:hover{ transform:translateY(-3px); border-color:rgba(150,170,220,0.4); background:rgba(30,38,62,0.85); }
a.card .emoji{ font-size:33px; display:block; margin-bottom:12px; }
a.card .go{ position:absolute; top:16px; right:16px; color:var(--muted); }
a.card .accent{ position:absolute; left:0; top:0; bottom:0; width:3px; }  /* 섹션 색 */
```

### 교과 단원 배지

3종. 문구는 **학년 + 수준 + 『단원명』**. 한 카드에 중복(기초+심화) 표시 가능.

```css
.badge{ font-size:12px; padding:2px 9px; border-radius:20px; border:1px solid; white-space:nowrap; }
.badge.basic{ background:rgba(93,202,165,0.14); color:#9be7c4; border-color:rgba(93,202,165,0.32); }  /* 중학 기초 */
.badge.adv{   background:rgba(239,159,39,0.13); color:#ffd479; border-color:rgba(239,159,39,0.32); }  /* 고등 심화 */
.badge.fusion{background:rgba(150,170,220,0.12); color:#bcc7e4; border-color:rgba(150,170,220,0.26); } /* 교과 밖 */
```

문구 예: `중2 기초 『전기와 자기』`, `중1 기초 『빛과 파동』`, `물리학Ⅱ 『평면 운동』`,
`화학Ⅰ 『원자의 세계』`, `생명과학Ⅰ 『생태계』`, `융합 · 교과 밖`.

### 푸터(피드백 연락처)

```html
<footer>
  각 시뮬레이션은 외부 라이브러리 없이 동작하는 단일 HTML 파일입니다.<br>
  카드를 누르고, 화면 안에서 드래그·클릭·슬라이더로 자유롭게 실험해 보세요.
  <div style="margin:24px auto 0; text-align:left; background:rgba(22,28,46,0.6); border:0.5px solid var(--border); border-radius:14px; padding:18px 22px; max-width:540px;">
    <p style="font-size:14px; color:var(--dim); margin:0 0 12px; line-height:1.65;">과학적 오류나 고쳐야 할 내용을 발견하면 아래로 편하게 알려주세요. 함께 더 좋은 학습 자료로 다듬어 가요.</p>
    <div style="font-size:14px; color:var(--dim); line-height:2.0;">
      📧 이메일 — <a href="mailto:이메일주소" style="color:#7fd4ef; text-decoration:none;">이메일주소</a><br>
      💬 오픈채팅 — <a href="오픈채팅URL" target="_blank" rel="noopener" style="color:#7fd4ef; text-decoration:none;">오픈카톡으로 문의하기 ↗</a>
    </div>
  </div>
</footer>
```

---

## 2. 마우스 따라오는 3D 로봇 배경 (허브 전용·선택)

허브에 생동감을 주려면 Spline의 로봇 씬을 **고정 배경**으로 깔고, 카드 밖 빈 영역에서 마우스가
로봇까지 통과하게 한다. 이것만 추가하고 다른 랜딩 요소(네비·문구·CTA)는 넣지 않는다.

> 바닐라 허브에는 React용 `@splinetool/react-spline`이 아니라 **`spline-viewer` 웹 컴포넌트**를 쓴다
> (빌드 불필요). React 프로젝트라면 별도로 `@splinetool/react-spline` + `<Suspense lazy>`를 쓴다.

### 핵심 원리

- 로봇은 `position:fixed` 전체화면 배경(`z-index:0`). 그 위에 어두운 스크림(가독성용,
  `pointer-events:none`), 그 위에 콘텐츠 `.wrap`(`z-index:1`).
- **마우스 추적의 비밀**: `.wrap`을 `pointer-events:none`으로, **카드·링크만 `auto`**로 둔다 →
  헤더·여백·카드 사이 빈 곳에서는 포인터가 로봇 캔버스까지 도달해 커서를 따라 반응하고,
  카드 위에서는 카드가 클릭을 받는다.
- 로봇 캔버스는 `spline-viewer`의 **shadow DOM** 안에 생긴다(`sv.shadowRoot.querySelector('canvas')`).

### CSS 추가

```css
#robotBg{ position:fixed; inset:0; z-index:0; }
#robotBg spline-viewer{ width:100%; height:100%; display:block; }
#robotScrim{ position:fixed; inset:0; z-index:0; pointer-events:none;
  background:radial-gradient(120% 90% at 50% 0%, rgba(7,10,18,0.25) 0%, rgba(7,10,18,0.55) 55%, rgba(7,10,18,0.82) 100%); }
.wrap{ position:relative; z-index:1; max-width:1080px; margin:0 auto; pointer-events:none; }
.wrap a{ pointer-events:auto; }   /* 카드·링크만 클릭 가능, 그 외 영역은 로봇이 마우스 추적 */
```

### HTML 추가

`<body>` 바로 다음:

```html
<div id="robotBg"><spline-viewer url="https://prod.spline.design/kZDDjO5HuC9GJUM2/scene.splinecode"></spline-viewer></div>
<div id="robotScrim"></div>
```

`</body>` 직전:

```html
<script type="module" src="https://unpkg.com/@splinetool/viewer@1.9.82/build/spline-viewer.js"></script>
```

### 검증 포인트

- `customElements.get('spline-viewer')` 정의됨 + `sv.shadowRoot`에 `<canvas>` 1개.
- `document.elementFromPoint`: 헤더·빈 영역 → `spline-viewer`, 카드 중앙 → 카드 내부 요소(클릭 가능).
- 콘솔 에러 0. (스크린샷이 헤드리스에서 막혀도 위 DOM 검사로 확정 가능.)
- 로봇은 외부 CDN(`prod.spline.design`)에서 로드되므로 인터넷 연결 필요.

> 참고: React + Vite + Tailwind로 풀 랜딩페이지(네비·히어로 문구·CTA·로봇 우측 배치)를 만들 수도
> 있다. 그때는 `pointer-events-none` 오버레이로 마우스를 로봇까지 통과시키고 버튼만 `auto`로 둔다.
