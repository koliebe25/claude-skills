# 프로바이더 연동

Gemini와 OpenAI의 차이를 한 곳에서 흡수해, 위쪽 코드가 어느 회사를 쓰는지 모르게 만듭니다.

> **먼저 확인하세요.** 모델 ID와 API 형태는 자주 바뀝니다. 아래는 2026-08 기준으로
> 실제 동작을 검증한 것입니다. 구현 전에 `node_modules/<sdk>/**/*.d.ts`를 열어
> 현재 SDK가 받는 파라미터를 확인하세요. 문서보다 정확합니다.

## 모델

| | 분석 (비전) | 이미지 생성 |
|---|---|---|
| Gemini | `gemini-3.7-flash` | `gemini-3-pro-image` (2K) / `gemini-3.1-flash-image` (1K) |
| OpenAI | `gpt-5.6-terra` | `gpt-image-2` |

모델 ID는 **환경변수로 빼두세요.** 정책 변경 시 코드 수정 없이 교체할 수 있습니다.

## 공통 인터페이스

```ts
export type Provider = "gemini" | "openai";

export interface Credentials {
  provider: Provider;
  apiKey?: string;   // 사용자 입력. 없으면 서버 환경변수 사용
}

analyzeOutfit(creds, imageDataUrl): Promise<string>   // JSON 문자열
generateImage(creds, { prompt, portrait, quality, sourceImage }): Promise<string>  // data URL
```

### 키 해석

사용자 키가 오면 그것을 쓰고, 없으면 서버 내장 키로 넘어갑니다.
**사용자 키는 어디에도 저장하지 않고 해당 요청 처리에만 씁니다.**

```ts
function resolveKey({ provider, apiKey }) {
  const trimmed = apiKey?.trim();
  if (trimmed) return trimmed;
  if (provider === "gemini") {
    const builtin = process.env.GEMINI_API_KEY;
    if (builtin) return builtin;
    throw new KeyError("내장 Gemini 키가 없습니다. 설정에서 본인 키를 입력해 주세요.");
  }
  throw new KeyError("OpenAI 를 쓰려면 본인 키를 입력해야 합니다.");
}
```

## Gemini

```ts
import { GoogleGenAI } from "@google/genai";
const ai = new GoogleGenAI({ apiKey });
```

**분석** — 구조화 출력으로 파싱 실패를 없앱니다.

```ts
const r = await ai.interactions.create({
  model: MODELS.gemini.analysis,
  input: [
    { type: "text", text: ANALYSIS_SYSTEM },
    { type: "text", text: "Analyze the outfit in this photo and return the breakdown as JSON." },
    { type: "image", mime_type: mimeType, data: base64 },
  ],
  response_format: { type: "text", mime_type: "application/json", schema: ANALYSIS_SCHEMA },
});
r.output_text  // JSON 문자열
```

**이미지 생성**

```ts
const r = await ai.interactions.create({
  model: quality === "pro" ? MODELS.gemini.imagePro : MODELS.gemini.imageStandard,
  input: [
    { type: "text", text: prompt },
    { type: "image", mime_type: mimeType, data: base64 },   // 참조 사진
  ],
  response_format: {
    type: "image",
    aspect_ratio: portrait ? "2:3" : "1:1",
    image_size: quality === "pro" ? "2K" : "1K",
  },
});
r.output_image  // { mime_type, data }
```

**JPEG만 반환합니다.** 알파 채널이 없으니 투명 배경을 요청하지 마세요 (`pitfalls.md` 함정 1).

## OpenAI

```ts
import OpenAI, { toFile } from "openai";
const client = new OpenAI({ apiKey });
```

**분석**

```ts
const r = await client.responses.create({
  model: MODELS.openai.analysis,
  input: [{
    role: "user",
    content: [
      { type: "input_text", text: `${ANALYSIS_SYSTEM}\n\n${INSTRUCTION}` },
      { type: "input_image", image_url: imageDataUrl, detail: "high" },
    ],
  }],
  text: { format: { type: "json_schema", name: "outfit_breakdown", schema: ANALYSIS_SCHEMA } },
});
r.output_text
```

**OpenAI 의 json_schema 는 사실상 항상 strict 로 동작합니다.** 공유 스키마를 그대로 넘기면
이 오류가 납니다.

```
Invalid schema for response_format 'outfit_breakdown':
In context=(), 'additionalProperties' is required to be supplied and to be false.
```

**모든 object 노드**에 `additionalProperties: false` 가 있어야 하고, 모든 속성이
`required` 에 들어 있어야 합니다. Gemini 는 반대로 `additionalProperties` 를 거부하므로,
공유 스키마는 건드리지 말고 **OpenAI 로 보낼 때만 변환**하세요.

```ts
function toOpenAiSchema(node: unknown): unknown {
  if (Array.isArray(node)) return node.map(toOpenAiSchema);
  if (!node || typeof node !== "object") return node;

  const src = node as Record<string, unknown>;
  const out: Record<string, unknown> = {};
  for (const [key, value] of Object.entries(src)) out[key] = toOpenAiSchema(value);

  if (src.type === "object" && src.properties && typeof src.properties === "object") {
    out.additionalProperties = false;
    out.required = Object.keys(src.properties as Record<string, unknown>);
  }
  return out;
}
```

중첩된 object 를 놓치기 쉬우니 **재귀로** 처리하세요. 이 앱의 스키마에는 루트·person·
items[] ·attributes 네 곳에 object 가 있습니다. `default` 필드도 strict 에서 거부됩니다.

**이미지 생성** — 참조 사진이 있으므로 `generate`가 아니라 `edit`을 씁니다.

```ts
const file = await toFile(Buffer.from(base64, "base64"), "reference.jpg", { type: mimeType });
const r = await client.images.edit({
  model: MODELS.openai.image,
  image: file,
  prompt,
  size: portrait ? "1024x1536" : "1024x1024",
  quality: quality === "pro" ? "high" : "medium",
  background: "opaque",     // gpt-image-2 는 transparent 를 거부한다
  output_format: "png",
});
r.data[0].b64_json
```

## 동시 실행

한 번에 3개씩 순차 생성해 속도 제한을 피합니다. 전부 동시에 던지면 429가 납니다.

```ts
async function pool(tasks, limit = 3) {
  const queue = [...tasks];
  await Promise.all(
    Array.from({ length: Math.min(limit, queue.length) }, async () => {
      while (queue.length) await queue.shift()();
    }),
  );
}
```

## 데모 모드

키가 없으면 자리표시 데이터로 전체 흐름이 돌아가게 하세요. 돈을 쓰지 않고 UI·레이아웃·
내보내기를 확인할 수 있고, 공개 배포 시 방문자의 기본 경험이 됩니다.

```ts
if (!apiKey?.trim() && provider === "gemini" && !hasBuiltinKey()) {
  return NextResponse.json({ ...mockAnalysis(), demo: true });
}
```

배포 후 **키 없는 상태를 반드시 실제로 확인하세요.** 로컬에서 환경 파일을 잠시 옮기고
서버를 재시작하면 배포 직후와 같은 상태가 재현됩니다.

## 사용자 키 보관

`sessionStorage`에만 둡니다. 탭을 닫으면 사라져 공용 PC에 남지 않습니다.
`localStorage`는 무기한 남으니 API 키에는 부적절합니다.

첫 렌더에서 읽으면 hydration 오류가 납니다 — `pitfalls.md` 함정 5를 반드시 읽으세요.
