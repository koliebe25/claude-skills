# 실제로 무너진 지점

전부 실제 개발에서 겪고 고친 것들입니다. 문서만 읽어서는 알 수 없고, 모르고 시작하면
같은 곳에서 반드시 막힙니다. **이미지 생성 단계에 들어가기 전에 읽으세요.**

---

## 함정 1 — 투명 배경을 요청하면 체크무늬를 그려버린다

**증상.** 프롬프트에 "transparent background"라고 썼더니 생성된 9장 전부에
포토샵의 회색-흰색 격자무늬가 **그림으로 그려져** 있었습니다.

**원인.** Gemini 이미지 API는 **JPEG만 반환**합니다. JPEG에는 알파 채널이 없어 투명도가
불가능합니다. 모델은 "투명"이라는 개념을 표현하려고 이미지 편집기의 투명 표시 패턴을
문자 그대로 그렸습니다. OpenAI `gpt-image-2`도 `background: "transparent"`를 거부합니다.

**해결.** 배경을 **순백(#FFFFFF)** 으로 요청하고, 체크무늬를 명시적으로 금지합니다.
알파는 클라이언트에서 직접 만듭니다.

```
- Fill the entire background with ONE flat pure white (#FFFFFF). Nothing else.
- Never draw a checkerboard, a grey-and-white tile grid, or any repeating square pattern.
  That pattern is a transparency placeholder from image editors and must never appear.
```

**교훈.** API가 무엇을 반환하는지 **SDK 타입 정의를 직접 열어** 확인하세요. 문서보다 정확합니다.
`node_modules/<sdk>/**/*.d.ts`를 grep하면 허용된 값이 그대로 나옵니다.

---

## 함정 2 — 배경을 지우면 흰 옷이 같이 사라진다

**증상.** 순백 배경으로 바꾸자 새 문제가 생겼습니다. 흰 데님·흰 크롭탑·흰 스니커즈를 입은
착장에서 배경 제거가 옷까지 먹을 위험이 생겼습니다. 실사로 가면 더 심해집니다 —
3D 렌더는 옷 색이 또렷하지만, 실사 흰 천은 배경과 RGB 차이가 20 안쪽입니다.

**해결 — 구조로 없앱니다.** 조건을 조정하는 대신, 위험한 경로 자체를 제거했습니다.

| 컷 | 처리 | 이유 |
|---|---|---|
| 제품컷 | **여백만 잘라냄** (배경 제거 안 함) | 어차피 포스터의 흰 카드 위에 놓임 |
| 착용컷 | 배경 제거 | 리넨 질감 배경 위에 서야 함 |

제품컷은 배경을 지울 이유가 없습니다. 흰 카드 위에 흰 배경 이미지를 얹으면 경계가 안 보입니다.
**옷이 지워질 경로 자체가 사라집니다.**

**프롬프트로도 보강합니다.**

```
If the item itself is white or near-white, let its natural fabric shading and a soft
contact shadow under its edges keep the silhouette clearly readable against the white.
```

착용컷에는 "흰 옷은 배경보다 몇 단계 어둡게 셰이딩하라"를 넣습니다.

**검증 방법.** 추측하지 말고 픽셀을 세세요. 테두리에서 플러드필을 돌린 뒤,
옷 내부 좌표가 제거됐는지 확인합니다. 실제 검증에서 배경만 75% 제거되고
흰 바지·크롭탑·스니커즈는 전부 보존됐습니다. 가로 한 줄을 문자열로 찍어보면
피사체가 구멍 없는 덩어리인지 한눈에 보입니다.

```
y=0.65  ........................################.....................
y=0.93  ..........................#####..#####.......................  ← 신발 두 짝
```

---

## 함정 3 — 429는 세 가지가 섞여 있다

같은 `429`인데 원인이 전혀 다릅니다. 구분하지 않으면 **기다려도 안 풀리는 오류를
계속 재시도하며** 시간을 버립니다.

| 종류 | 원문 메시지 | 기다리면 | 해야 할 일 |
|---|---|---|---|
| 분당 요청 초과 | (일반 rate limit) | **풀림** | 지수 백오프 재시도 |
| 무료 등급 쿼터 0 | `free_tier`, `limit: 0` | 안 풀림 | 결제 연결 |
| 월 지출 한도 초과 | `spending cap` | 안 풀림 | 콘솔에서 한도 상향 |
| 크레딧 부족 | `insufficient_quota` | 안 풀림 | 결제 확인 |

**구현.**

```ts
function quotaKind(error) {
  if (status !== 429) return null;
  if (/spending cap|spend cap/i.test(message)) return "spendCap";
  if (/free_tier|limit: 0/i.test(message)) return "freeTier";
  if (/insufficient_quota|exceeded your current quota/i.test(message)) return "insufficientCredit";
  return null;  // 분당 초과 → 재시도 대상
}
```

`quotaKind`가 값을 반환하면 **재시도를 건너뜁니다.** 이 처리 하나로 실패 응답이
35초에서 11초로 줄었습니다.

> **이미지 생성은 무료 등급에서 쓸 수 없습니다.** 실측 결과 `limit: 0`으로 완전히 막혀
> 있고, 레거시 모델을 포함해 우회 경로가 없습니다. 텍스트 분석은 무료로 됩니다.
> "무료로 하루 500장" 같은 오래된 정보가 웹에 많으니 믿지 마세요.

---

## 함정 4 — SDK가 진짜 오류를 숨긴다

**증상.** 오류 메시지가 이랬습니다.

```
API error occurred: {"httpMeta":{"response":{},"request":{}}}
```

아무 정보가 없습니다. 진짜 사유는 `error.body`에 JSON 문자열로 따로 들어 있었습니다.

```json
[{"error":{"code":400,"message":"API key not valid. Please pass a valid API key.",
  "status":"INVALID_ARGUMENT"}}]
```

**해결.** 오류 메시지를 읽을 때 `body`를 먼저 파싱합니다.

```ts
function messageOf(error) {
  if (typeof error.body === "string") {
    try {
      const parsed = JSON.parse(error.body);
      const first = Array.isArray(parsed) ? parsed[0] : parsed;
      const detail = first?.error?.message ?? first?.message;
      if (detail) return String(detail);
    } catch {}
  }
  return error.error?.message ?? error.message ?? "";
}
```

**Gemini는 잘못된 키를 401이 아니라 400 `INVALID_ARGUMENT`로 반환합니다.**
상태 코드만 보고 분기하면 놓칩니다. 메시지 패턴도 함께 봐야 합니다.

```ts
if (status === 401 || status === 403 ||
    /API key not valid|API_KEY_INVALID|Incorrect API key/i.test(message)) { ... }
```

**교훈.** 오류가 의미 없어 보이면 **오류 객체 전체를 서버에서 찍어보세요.**
라이브러리가 중요한 정보를 다른 칸에 넣어두는 일이 흔합니다.

---

## 함정 5 — 저장된 키를 첫 렌더에서 읽으면 화면이 깨진다

**증상.** 사용자가 API 키를 한 번이라도 입력하면, 이후 새로고침마다
`Hydration failed because the server rendered text didn't match the client` 오류가 났습니다.

**원인.** 상태 초기값을 `sessionStorage`에서 읽었습니다. 서버에는 저장소가 없어 "기본 키"로
렌더하고, 브라우저는 저장된 값을 읽어 "내 OpenAI 키"로 렌더합니다. **같은 자리의 글자가
달라집니다.**

**해결.** 첫 렌더는 빈 값으로 두고, 마운트 이후에 읽습니다.

```ts
keys: EMPTY_KEYS,        // 서버와 첫 클라이언트 렌더가 같아야 한다
keysHydrated: false,
hydrateKeys: () => {
  if (get().keysHydrated) return;   // 한 번만
  set({ keys: loadKeys(), keysHydrated: true });
},
```

```tsx
useEffect(() => { hydrateKeys(); }, [hydrateKeys]);
```

**"한 번만"이 중요합니다.** 시크릿 모드처럼 저장이 막힌 환경에서 컴포넌트가 재마운트될 때
입력하던 키가 날아가는 것을 막습니다.

**주의.** 콘솔 버퍼는 누적이라 수정 전 오류와 섞입니다. 정확히 재현하려면
**새 탭**에서 키를 심고 새로고침한 뒤 그 탭의 콘솔만 보세요.

---

## 함정 6 — 개발 서버를 켠 채로 빌드하지 말 것

**증상.** `Cannot find module './331.js'` 로 실행 중이던 개발 서버가 깨졌습니다.
CSS가 404를 내고 `onUnhandledRejection`까지 이어져 코드 문제로 착각했습니다.

**원인.** 빌드가 `.next` 폴더를 덮어써서 실행 중인 서버의 청크 참조가 끊어졌습니다.

**해결.** 개발 서버를 **끄고** 빌드하세요. 이미 깨졌다면 서버 중지 → `.next` 삭제 →
재시작하면 됩니다.

---

## 함정 7 — 비동기 실패를 삼키면 개발 오버레이가 화면을 덮는다

`onClick={() => void handleExport()}` 처럼 `void`로 호출하면서 `catch`가 없으면,
실패 시 unhandled rejection이 되어 Next 개발 오버레이가 화면 전체를 덮습니다.

사용자에게는 배너로 알리고 앱은 계속 쓸 수 있게 두세요.

```ts
try {
  await exportPoster(...);
} catch (err) {
  setExportError(err instanceof Error ? err.message : "PNG 내보내기에 실패했습니다.");
} finally {
  setExporting(false);
}
```

---

## Windows에서 검증할 때

**PowerShell은 빈 문자열 인자를 삭제합니다.** `node test.mjs a "" b` 로 넘기면 `""`가
사라져 뒤 인자가 한 칸씩 밀립니다. 실제로 이 때문에 테스트가 엉뚱한 문자열을 API 키로
전송해 "API key not valid"를 받았고, 앱 버그로 오인해 디버깅 사이클을 낭비했습니다.
빈 값 대신 `-` 같은 센티널을 쓰세요.

**PowerShell로 한글 파일을 재작성하지 마세요.**
`(Get-Content f -Raw) -replace ... | Set-Content -Encoding utf8` 는 한글 문서를 전부
모지바케로 깨뜨립니다. 읽기 단계에서 ANSI로 해석되기 때문입니다. 파일 수정은 편집 도구만 쓰세요.

**git stderr는 오류가 아닙니다.** PowerShell이 native 명령의 stderr를 `NativeCommandError`로
감싸서 성공한 push도 실패처럼 보입니다. 실제 결과는 `git status -sb`로 확인하세요.
