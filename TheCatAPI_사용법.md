# The Cat API 사용법 정리

> 출처: https://thecatapi.com/ · https://docs.thecatapi.com/ · OpenAPI 스펙 https://api.thecatapi.com/openapi-json (v1.7.1)
> 조회일: 2026-11-09

## 1. 기본 정보

| 항목 | 내용 |
|---|---|
| Base URL | `https://api.thecatapi.com/v1` |
| 인증 | 요청 헤더 `x-api-key: YOUR-API-KEY` |
| API 키 발급 | https://thecatapi.com/signup (이메일로 발송), 계정 포털 https://account.thecatapi.com |
| 응답 형식 | JSON |

### 요금제
| 플랜 | 가격 | 요청 한도 | 비고 |
|---|---|---|---|
| Free | $0 | 월 10,000건 | 품종 핵심 필드(이름, 성격, 원산지, 수명, 설명, 이미지) |
| Startup | $99/월 | 월 100,000건 | 상업적 사용 라이선스, 품종 점수·특성·facts, 현지화(`lang`) |
| Enterprise | 협의 | 무제한 | 건강 팁, 사진 분석, 초상화, 계산기 등 |

- `GET /accounts/features`로 내 키에서 쓸 수 있는 기능을 확인할 수 있다.
- 상태 코드·재시도 기준은 문서의 "Errors and limits" 페이지에 있다. 이번 조회에서는 상세 내용을 확인하지 못했다.

## 2. 키 없이 쓸 수 있는 엔드포인트
스펙상 아래 4개는 `x-api-key` 없이 호출할 수 있다. 현재 `index.html`도 키 없이 `/images/search`를 쓴다.
- `GET /images/search`
- `GET /images/{id}`
- `GET /votes`
- `GET /votes/{vote_id}`

키 없이 쓰면 한도 등이 제한될 수 있으므로, 실서비스에서는 키를 쓰는 편이 안전하다. (`limit`는 키 사용 시 1–100)

## 3. 이미지 (Images)

### 이미지 검색 `GET /images/search`
```bash
# 랜덤 1장 (키 없이도 가능)
curl "https://api.thecatapi.com/v1/images/search"

# 2장
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/images/search?limit=2"

# 특정 품종 + 품종 정보 포함
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/images/search?limit=2&breed_ids=abys&include_breeds=1"
```

응답은 이미지 객체 배열이며 `id`, `url`, `width`, `height`, `breeds` 필드를 가진다.

| 파라미터 | 설명 |
|---|---|
| `limit` | 반환 개수 (1–100, 기본 1) |
| `page` | 페이지 번호 (0부터) |
| `order` | `ASC` / `DESC` / `RAND` (스펙상 `RANDOM`도 표기됨) |
| `has_breeds` | `true`면 품종 정보가 있는 이미지만 |
| `include_breeds` | 품종 객체 포함 |
| `include_categories` | 카테고리 객체 포함 |
| `breed_id` / `breed_ids` | 품종 ID 필터 (`breed_ids`는 쉼표 구분) |
| `mime_types` | `jpg`, `png`, `gif` (쉼표 구분) |
| `size` | `thumb` / `small` / `med` / `full` |
| `sub_id` | 해당 `sub_id`로 업로드한 이미지만 |
| `colour_id(s)`, `colour_code`, `has_colours`, `include_colours` | 색상 필터 |
| `age_months_min`, `age_months_max` | 나이(개월) 범위 |
| `account_id`, `order_by`, `species_id`, `breed_groups`, `exclude_breed_groups`, `lang` | 기타 필터 |

### 기타 이미지 API
| 메서드 | 경로 | 설명 |
|---|---|---|
| GET | `/images/{id}` | 이미지 1건 조회 |
| GET | `/images` | 내가 업로드한 이미지 목록 (`?sub_id=` 필터 가능) |
| POST | `/images/upload` | 비동기 업로드 (`type`=base64\|FILE, `file` 또는 `image`, `sub_id`) |
| POST | `/images/upload-sync` | 업로드 후 분석 완료까지 대기 (`auto_crop` 기본 true) |
| GET | `/images/{id}/status` | 처리 상태 확인 (분석 끝날 때까지 폴링) |
| GET | `/images/{id}/labels` | 품종·카테고리 AI 라벨 |
| DELETE | `/images/{id}` | 업로드 이미지 삭제 |

## 4. 품종 (Breeds)
```bash
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/breeds                    # 전체 목록
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/breeds/search?q=siamese" # 검색
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/breeds/abys               # 상세
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/breeds?breed_groups=Short-haired,Rex"
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/breeds?lang=it"           # 현지화
```

| 경로 | 설명 |
|---|---|
| `GET /breeds` | 목록 (`limit` 1–1000, `page`, `extinct`, `venom_code`, `breed_groups`, `lang` 등) |
| `GET /breeds/search?q=` | 검색 (`q` 1–30자) |
| `GET /breeds/{id}` | 상세 |
| `GET /breeds/breed-groups` | 품종 그룹 목록 |
| `GET /breeds/{id}/colours` | 품종의 색상·무늬 |

- 품종 ID는 4글자 코드다. 예: `abys`(아비시니안), `beng`(벵갈), `siam`(샴).
- 응답 예시(일부):
```json
{
  "id": "abys",
  "name": "Abyssinian",
  "life_span": "14-17",
  "temperament": "Active, Energetic, Independent, Intelligent, Gentle, Curious, Playful",
  "origin": "Egypt",
  "breed_group": "Short-haired",
  "image": { "id": "KWdLHmOqc", "url": "https://cdn2.thecatapi.com/images/KWdLHmOqc.jpg", "width": 3114, "height": 2609 }
}
```
- 1–5 점수(애정, 에너지 등)와 불리언 특성(hypoallergenic 등)은 Startup 이상에서 제공된다.

### 품종 규제 (Breed Bans)
- `GET /breeds/bans/countries`, `GET /breeds/bans/country?country=KR`, `GET /breeds/bans/breed/{breedId}`

## 5. Facts (고양이 상식, Startup 이상)
| 경로 | 설명 |
|---|---|
| `GET /facts` | 목록 (`limit` 1–100, `offset`, `breed_id`, `species_id`, `category`, `include_sources`, `lang`) |
| `GET /facts/random` | 랜덤 1건 |
| `GET /facts/breed/{breedId}` | 품종별 목록 (`?limit=5`) |
| `GET /facts/breed/{breedId}/random` | 품종별 랜덤 |
| `GET /facts/species/{speciesId}/random` | 종별 랜덤 |
| `GET /facts/{id}` | ID로 조회 |

## 6. 투표 (Votes)와 즐겨찾기 (Favourites)
```bash
# 투표 (value: 1 = 좋아요, -1 = 싫어요)
curl -X POST -H "Content-Type: application/json" -H "x-api-key: YOUR-API-KEY" \
  -d '{"image_id": "KWdLHmOqc", "value": 1}' https://api.thecatapi.com/v1/votes

# 즐겨찾기 추가 / 목록 / 삭제
curl -X POST -H "Content-Type: application/json" -H "x-api-key: YOUR-API-KEY" \
  -d '{"image_id": "KWdLHmOqc"}' https://api.thecatapi.com/v1/favourites
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/favourites
curl -X DELETE -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/favourites/FAVOURITE_ID
```
- 투표 body: `image_id`(필수), `value`(필수, 1 또는 -1), `sub_id`, `country_code`, `ip_address`
- **투표는 영구 기록이라 삭제할 수 없다.** 취소하려면 반대 `value`로 다시 투표한다.
- `sub_id`를 보내면 내 서비스 사용자별로 기록하고 `?sub_id=`로 필터할 수 있다.
- 즐겨찾기 목록: `limit`, `page`, `order`, `image_id`, `sub_id`, `attach_image` 등을 지원한다.
- 데모: https://examples.thecatapi.com/voting/ (소스: GitHub `thatapicompany/thecatapi-examples`)

## 7. 그 외 엔드포인트 (요약)
- **Categories**: `GET /categories`, `GET /categories/{id}`
- **Health Tips** (Enterprise): `GET /health-tips`, `GET /health-tips/breed/{breedId}`, `GET /health-tips/{id}`
- **Pets** (Enterprise 기능 포함): `/pets` 등록·조회·수정·삭제, 사진 분석(체형 점수, 나이·체중 추정, 혈통 분석), 초상화 생성, 방문 기록, 관찰 기록
- **Calculators**: `POST /calculators/energy-requirement`, `POST /calculators/weight-loss-plan`
- **Webhooks**: `POST/GET /webhooks`, `GET /webhooks/topics`, `DELETE /webhooks/{id}`, `POST /webhooks/{id}/rotate-secret`
- **Accounts**: `GET /accounts/features`, `/accounts/quota`, `/accounts/capabilities`
- **Sub IDs**: `GET /sub-ids` (sub_id 사용 현황)

## 8. JavaScript 사용 예시
```js
const API_KEY = "YOUR-API-KEY"; // 실서비스에서는 노출되지 않도록 서버에서 호출하는 것을 권장

async function getCats(limit = 3) {
  const res = await fetch(
    `https://api.thecatapi.com/v1/images/search?limit=${limit}&has_breeds=true&include_breeds=1`,
    { headers: { "x-api-key": API_KEY } }
  );
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json(); // [{ id, url, width, height, breeds: [...] }, ...]
}
```

## 9. 참고
- 정리 내용은 공식 문서와 OpenAPI 스펙에서 가져왔다. 스펙 일부(POST/PATCH 본문 스키마 등)는 조회 범위를 넘어 확인하지 못했다. 자세한 내용은 https://docs.thecatapi.com/reference 를 참고한다.
- 커뮤니티: Discord https://discord.gg/UZTWaEVYP5
