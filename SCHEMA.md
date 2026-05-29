# 스냅샷 스키마 (schema_version: 1)

rs-golden-queens 봇의 fetcher 반환 구조를 그대로 발행 payload로 사용한다.
색·이모지는 저장하지 않는다(소비 측이 `pct`/부호로 계산).

## 공통 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `schema_version` | int | 현재 `1` |
| `market` | string | `"kr"` \| `"us"` \| `"weekly"` |
| `date` | string | timezone-aware ISO 날짜. 시장별 거래일 (KR=KST, US=미국 거래일) |
| `generated_at` | string | 발행 시각 (ISO, +09:00) |
| `is_holiday` | bool | 휴장 여부 |
| `payload` | object\|null | 데이터 본문. 휴장 시 `null` |
| `sources` | array | `{label, url}` 출처 목록 |

휴장 스냅샷은 추가로 `message`(string)를 가진다.

## KR — `snapshots/kr/<YYYY-MM-DD>.json`

`payload`:

- `bizdate`: string (YYYYMMDD)
- `kospi`, `kosdaq`: `{ personal, foreign, institutional, program_arb, program_nonarb, program_total }` (단위: 억원, 순매수)
- `kospi_daily`: 최대 10거래일 배열. 각 행:
  `{ date, personal, foreign, institutional, finance, insurance, trust, bank, other_fin, pension, other_corp }`

> `kospi_intraday`(장중 분단위)는 아카이브에 불필요하므로 발행하지 않는다.

## US — `snapshots/us/<YYYY-MM-DD>.json`

`payload`의 각 섹션은 `{티커: {label, close, pct, vol_ratio, date}}` 형태로 통일:

| 섹션 | 항목 수 | 비고 |
|------|---------|------|
| `indices` | 4 | S&P500·나스닥·다우·러셀2000 |
| `volatility` | 3 | VIX·VVIX·SKEW (`vol_ratio` null 가능) |
| `risk_onoff` | 2 | HYG·IEF |
| `macro` | 6 | 금리·DXY·원달러·WTI·금 |
| `sectors` | 11 | S&P 11섹터 |
| `watch` | 8 | 워치 ETF (`vol_ratio` = 거래량강도, 당일/5일평균) |

## Weekly — `snapshots/weekly/<YYYY-Www>.json`

추가 필드 `week`: ISO 주차 (예: `"2026-W22"`).

`payload`:
- `kospi_daily`: KR과 동일 구조 (5~10행)
- `watch_5d`: `[{ ticker, pct_5d }]` (8개)

## index.json

```jsonc
{
  "schema_version": 1,
  "updated_at": "...",
  "kr":     ["2026-05-29", "2026-05-28", ...],
  "us":     ["2026-05-28", ...],
  "weekly": ["2026-W22", ...]
}
```

## latest.json

홈 요약. 각 시장 최신 스냅샷의 경로/날짜 + 핵심 요약 수치.
```jsonc
{
  "schema_version": 1,
  "updated_at": "...",
  "kr":     { "date": "2026-05-29", "path": "snapshots/kr/2026-05-29.json" },
  "us":     { "date": "2026-05-28", "path": "snapshots/us/2026-05-28.json" },
  "weekly": { "week": "2026-W22",   "path": "snapshots/weekly/2026-W22.json" }
}
```
