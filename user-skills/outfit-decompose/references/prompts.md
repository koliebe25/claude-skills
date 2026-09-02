# 프롬프트 전문

실제로 동작을 검증한 프롬프트입니다. 그대로 복사해 쓰고, 사용자 요구에 맞춰 조정하세요.

**전부 한 파일(`src/lib/prompts.ts`)에 모아두세요.** 실제 개발에서 화풍 요구사항이 두 번
뒤집혔는데(3D → 혼합 → 전부 실사), 프롬프트가 격리되어 있던 덕분에 화면·상태·API 코드를
하나도 건드리지 않고 이 파일만 고쳐서 끝났습니다.

---

## 1. FRAME_RULES — 착용컷과 제품컷이 공유

배경과 프레이밍을 일치시켜야 포스터에서 나란히 놓았을 때 정돈되어 보입니다.

```
FRAMING & BACKGROUND (identical for every image in this set):
- Camera straight-on, zero perspective distortion, no tilt, subject centered.
- Fill the entire background with ONE flat pure white (#FFFFFF). Nothing else.
- Never draw a checkerboard, a grey-and-white tile grid, or any repeating square pattern.
  That pattern is a transparency placeholder from image editors and must never appear.
- No floor, no wall, no table, no surface, no backdrop, no scenery, no gradient,
  no vignette, no cast shadow on any surface.
- Absolutely no text, no watermark, no logo, no label, no caption, no measurement marks,
  no arrows, no border, no frame anywhere in the image.
```

> **체크무늬 금지 문구를 지우지 마세요.** "투명 배경"을 요청했더니 모델이 포토샵의 회색
> 격자무늬를 *그림으로 그려서* 반환한 실제 사고가 있었습니다. 자세한 내용은 `pitfalls.md` 함정 1.

---

## 2. PORTRAIT_STYLE — 착용컷 (실사 인물)

```
RENDER STYLE — photorealistic full-body fashion photograph:
- A real photograph of a real person. Real skin, real hair strands, real fabric with
  visible weave and natural drape. Never an illustration, never a 3D render, never
  a cartoon, never a doll or figurine, never CGI.
- Shot on a full-frame camera with a 50mm lens, sharp focus, natural depth,
  accurate exposure, neutral white balance.
- Lighting: soft even daylight-balanced studio light, gentle shadows, no harsh
  speculars, no coloured light, no heavy retouching.
- Colour: true to the garment colours in the reference photo.

{FRAME_RULES}
```

### 착용컷 프롬프트 조립

`person`(분석 결과)과 아이템 목록을 끼워 넣습니다.

```
A full-body mirror-selfie fashion photograph of one person wearing the outfit below.

{PORTRAIT_STYLE}

PERSON
{person.visualPrompt}
Hair: {person.hair}
Build: {person.bodyType}

POSE — this exact pose, regardless of the pose in the reference photo:
Standing straight and facing the camera, holding a smartphone up in front of their own
face with one hand at roughly eye level, taking a mirror selfie. The other arm rests
naturally at the side or with the hand in a pocket.

FACE — mandatory, this is a privacy requirement:
The raised smartphone must FULLY cover the face. Not one facial feature may be visible:
no eyes, no eyebrows, no nose, no mouth, no chin, no jawline, no cheeks. The phone is
held flat, parallel to the camera, large enough and positioned high enough to hide the
whole face from hairline to chin. Hair may frame the sides of the phone.
Do not reconstruct, reveal, peek around, or show a reflection of the face anywhere.
If the face would still be partly visible, enlarge the phone until it is fully hidden.

OUTFIT — the person wears exactly this, nothing added, nothing removed:
{각 아이템의 "- {label}: {visualPrompt}" 목록}

OUTPUT
- Full body, head to feet, entirely inside the frame with a small margin at top and bottom.
- Flat pure white (#FFFFFF) background as specified above. Keep the person clearly
  separated from it: a soft but definite outline everywhere, and never let a white
  garment bleed into the background — shade white garments a few steps darker than the
  background so their silhouette stays readable.
- One person only. No duplicates, no mirror frame, no visible reflection, no extra limbs.
```

**포즈를 원본과 무관하게 강제하는 것이 핵심입니다.** 원본 사진의 포즈를 따라가게 두면
얼굴이 드러납니다. "regardless of the pose in the reference photo"를 빼지 마세요.

**"그래도 얼굴이 보인다면 폰을 더 크게"** 같은 복구 지시가 실제로 효과가 있습니다.
모델에게 스스로 판단해 고칠 여지를 줍니다.

---

## 3. PRODUCT_STYLE — 제품컷 (실사 플랫레이)

```
RENDER STYLE — professional e-commerce product photography, photorealistic:
- A real photograph, not an illustration and not a 3D render. Real fabric with visible
  weave and surface texture, real stitching, real hardware with true metal reflections,
  natural material drape and weight.
- Sharp focus edge to edge, high detail, accurate exposure, neutral white balance.
- Lighting: large soft studio softbox from the front-upper-left plus fill, the even
  shadowless look of a catalogue shot. No harsh speculars, no coloured light.
- Colour accuracy is critical: the rendered colour must match the reference garment.

FIDELITY (this image will be used in real product advertising):
- Reproduce the actual product exactly as it appears in the reference photo.
- Do NOT redesign, restyle, modernise, embellish, or "improve" it. Do not add pockets,
  buttons, seams, prints, logos or trims that are not visible in the reference.
- Keep the pattern at its true scale and alignment, and keep proportions honest.

{FRAME_RULES}
```

> **FIDELITY 절은 법적 요구사항입니다.** 생성 이미지가 실물과 다르면 허위·과장 광고가 됩니다.
> "AI가 만들어서"는 면책 사유가 되지 않습니다. 미학이 아니라 규제의 문제라서, 사용자가
> "좀 더 예쁘게"를 요청해도 이 절은 유지하고 대신 조명·구도로 조정하세요.

### 제품컷 프롬프트 조립

```
A single flat-lay product photograph of one garment/accessory, shown alone with
nothing else in frame. No mannequin, no hanger, no person, no body parts,
no hands, no hair, no packaging, no props, no second item, no size chart.

{PRODUCT_STYLE}

ITEM — reproduce exactly, this is the only subject:
{item.visualPrompt}

REFERENCE
The attached photo shows this item being worn. Match its true color, pattern scale,
proportion and material. Reconstruct the parts hidden by the body or by other garments
so the item is shown complete and undistorted.

FRAMING
{아래 weight별 프레이밍}

OUTPUT
- Flat pure white (#FFFFFF) background as specified above. If the item itself is white or
  near-white, let its natural fabric shading and a soft contact shadow under its edges
  keep the silhouette clearly readable against the white.
- Exactly one subject. No collage, no grid, no variations, no alternate angles.
```

**weight별 프레이밍** — 작은 액세서리를 크게 채우면 뭉개집니다.

```
major:  Centered, filling about 88% of the frame, laid flat and shot from directly above
        the way a catalogue product photo would show it (tops laid open and symmetrical,
        bottoms laid flat with legs together, bags upright, shoes as a three-quarter pair).

minor:  Centered, filling about 70% of the frame, arranged neatly. If the item is
        naturally a pair or a small set, show the set together in one tidy arrangement.
```

**원본 사진을 참조 이미지로 함께 보내세요.** 텍스트 묘사만으로는 색·패턴이 어긋납니다.

---

## 4. ANALYSIS_SYSTEM — 착장 분석

```
You are a senior fashion merchandiser preparing a lookbook breakdown sheet.
You examine a single full-body photo of one person and decompose the outfit into
individually purchasable/renderable components.

RULES
1. Only list components you can actually SEE. Never invent an item. If there is no
   outerwear, do not emit a TOP (OUTER) entry. If the shoes are cropped out of frame,
   do not emit SHOES.
2. Merge tiny same-kind jewelry into one entry (e.g. three rings -> "RINGS").
3. NEVER emit an item for the hairstyle itself. Hair is not a product.
   Only when the person actually WEARS a hair product — a clip, pin, ribbon, headband,
   scrunchie, barrette — emit it as "HAIR ACCESSORY", describing only the object.
   A hat, cap or beanie is "HAT" instead. If there is no such product, emit neither.
4. Order items by visual importance: outerwear and main garments first, then bag and
   shoes, then small accessories.
5. Emit between 4 and 9 items. If you find more, merge the least important ones.
6. "weight" controls poster sizing:
   - "major": full garments (outer, inner top, bottom, dress, bag, shoes, hat)
   - "minor": small accessories (jewelry, eyewear, watch, hair clip, socks, belt)
7. visualPrompt must be an English description dense enough to redraw the item from
   scratch WITHOUT seeing the photo: silhouette, cut, length, closure, collar/neckline,
   sleeve type, exact color, pattern scale and layout, material and its sheen, hardware
   finish, stitching, and any distinguishing detail. 40-70 words. Describe ONLY the
   garment itself, never the person wearing it, never a background.
8. label is the caption printed on the poster. It MUST be exactly one of these strings,
   chosen as the closest fit, and must NOT be a descriptive phrase:
   HAIR ACCESSORY / HAT / TOP (OUTER) / TOP (INNER) / DRESS / BOTTOM / JEANS / SKIRT /
   BAG / SHOES / NECKLACE / EARRINGS / WATCH / RINGS / BELT / EYEWEAR / SOCKS /
   ACCESSORIES
   Write "TOP (OUTER)" for a shirt worn open over another top, "TOP (INNER)" for the
   layer underneath. Set category to the same string as label.
   Never write things like "PLAID SHIRT", "BOW HAIRPIN" or "WIDE JEANS" — the specific
   description belongs in attributes.nameKo, not on the poster.
9. attributes.nameKo is a natural Korean product name a Korean MD would write,
   e.g. "인디고 페이즐리 오버핏 가운 자켓".
10. colorHex must be the dominant color sampled from the actual garment.

PERSON PROFILE
Describe the person for a photorealistic full-body fashion photograph: hair
colour/length/texture and approximate build. Do NOT describe the face — the face will be
hidden behind a phone. Do not describe clothing in person.visualPrompt — the clothing is
handled separately. For "pose", just note the general standing posture. Never describe
skin tone with food metaphors; use plain neutral wording. Never guess age, ethnicity,
or identity.
```

### 규칙 8이 왜 이렇게 긴가

처음에는 "짧은 대문자 라벨"이라고만 썼더니 모델이 `PLAID SHIRT`, `BOW HAIRPIN`,
`WIDE JEANS` 같은 서술형을 반환했습니다. **허용 문자열을 열거하고 나쁜 예까지 적시**하고 나서야
`TOP (OUTER)`, `JEANS` 같은 카테고리 라벨로 고정됐습니다.

라벨은 포스터에 인쇄되므로 길이가 들쭉날쭉하면 레이아웃이 무너집니다. 구체적인 상품명은
`attributes.nameKo`로 보내 우측 속성 패널과 CSV에서 쓰세요.

### 구조화 출력 스키마

응답을 JSON 스키마로 강제하세요. 파싱 실패가 사라집니다.

```
title         포스터 제목 (영문 2~4단어)
moodKeywords  한글 스타일 키워드 3~5개
paletteHex    착장 대표색 5개
person        { visualPrompt, hair, bodyType, pose }
items[]       { label, category, weight, visualPrompt,
                attributes: { nameKo, colorName, colorHex, material, fit,
                              pattern, detail, styleKeyword } }
```
