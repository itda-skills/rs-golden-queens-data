# rs-golden-queens-data

[rs-golden-queens](https://github.com/itda-skills/rs-golden-queens) 봇이 발행하는 **시장 데이터 스냅샷 아카이브**.

매 거래일 마감 후, 텔레그램으로 발송한 것과 동일한 확정 데이터를 구조화 JSON으로 발행한다.
이 저장소는 **발행물(publish) 전용**이며, 봇의 운영 수집(raw) 데이터는 저장하지 않는다.
Vercel 웹([rs-golden-queens #1](https://github.com/itda-skills/rs-golden-queens/issues/1))이 이 데이터를 읽기 전용으로 소비한다.

## 디렉토리 구조

```
snapshots/
  kr/<YYYY-MM-DD>.json       한국장 일일 (코스피·코스닥 투자자별 + 프로그램매매 + 10거래일 추이)
  us/<YYYY-MM-DD>.json       미국장 일일 (지수·변동성·risk·매크로·섹터·워치ETF)
  weekly/<YYYY-Www>.json     주간 (코스피 누적 + 워치ETF 5일 등락)
  index.json                 발행된 전체 날짜 목록 + 메타
  latest.json                홈 요약 (최신 KR/US/주간)
```

## 스키마

- 버전: `schema_version: 1`
- 모든 스냅샷: `market`, `date`(timezone-aware), `is_holiday`, `payload`, `sources`, `generated_at`
- **색·이모지 문자열은 저장하지 않는다.** 값/부호만 저장하고, 색 컨벤션(🔴▲상승 / 🔵▼하락)은 소비 측(웹)이 계산한다.
- 휴장일: `is_holiday: true` + `message`, `payload: null`.

상세 스키마는 [SCHEMA.md](./SCHEMA.md) 참조.

## 발행 주체

- 발행: `rs-golden-queens` 코드 저장소의 GitHub Actions(`flow-kr`/`flow-us`/`flow-weekly`)가 deploy key로 이 저장소에 push.
- 이 저장소는 사람이 직접 커밋하지 않는다(봇 자동 발행 전용).
