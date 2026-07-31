# loadrunner_report

WhaTap APM **성능 추이 CSV**를 넣으면 부하테스트 리뷰용 **HTML 리포트**를 자동 생성하는 도구입니다.
부하 세션(회차)을 자동으로 탐지하고, 세션별 최대 TPS·응답시간·동시 Active Tx와 시간대별 추이 차트를 만들어 줍니다.

> 다음에 **날짜·시간대만 지정**하면 같은 형식의 리포트가 나오도록 만든 재사용 도구입니다.

## 요구사항

- Python 3.8+ (표준 라이브러리만 사용, 추가 설치 불필요)

## 빠른 시작

```bash
# 동봉된 합성 샘플로 바로 실행
python generate_report.py --csv sample_perf.csv --out sample_report.html
```

`sample_report.html` 을 브라우저로 열면 결과를 볼 수 있습니다.

## 사용법

```bash
python generate_report.py --csv <성능추이.csv> [옵션]
```

| 옵션 | 설명 | 기본값 |
|------|------|--------|
| `--csv` | WhaTap 성능추이 CSV 경로 (필수) | — |
| `--title` | 리포트 제목 | 부하테스트 결과 리포트 |
| `--out` | 출력 HTML 경로 | `<csv이름>_report.html` |
| `--start` / `--end` | 분석 구간 필터 (`"2026-07-27 10:00"`) | 전체 |
| `--notes` | 정성 분석 마크다운(.md) 첨부 | 없음 |
| `--tps-threshold` | 세션 탐지 TPS 임계 | 50 |
| `--max-gap-min` | 세션 병합 최대 간격(분) | 30 |

예시:

```bash
python generate_report.py --csv perf.csv \
  --title "수강신청 부하테스트" \
  --start "2026-07-27 10:00" --end "2026-07-27 17:00" \
  --notes 작업기록.md \
  --out report.html
```

## 1. 데이터 수집 (WhaTap)

1. WhaTap 로그인 → 대상 프로젝트
2. **애플리케이션 › 분석 › 성능 추이** 이동
3. 상단에서 **날짜·시간대 지정** 후 **파란 조회(돋보기) 버튼**을 눌러 차트가 해당 구간으로 갱신되는지 확인
   - (성능 추이 화면은 날짜 컨트롤이 실제 적용돼야 차트가 바뀝니다)
4. **CSV** 버튼 → 다운로드

내려받은 CSV 헤더 형식:

```
"Timestamp","Realtime User (count)","TPS","Response Time (ms)","CPU (%)","Heap (byte)","Active Tx (count)"
```

## 2. 리포트 생성

```bash
python generate_report.py --csv 내려받은.csv
```

## 세션 자동 탐지 방식

- `TPS >= tps-threshold`(기본 50)인 5분 버킷을 **활성**으로 판단
- 활성 버킷 사이 간격이 `max-gap-min`(기본 30분) 이하이면 **같은 세션**으로 병합
- 세션별로 최대 TPS / 최대 평균 응답시간 / 최대 동시 Active Tx / 피크 동시접속을 계산

부하 패턴에 맞춰 임계·간격을 조정하세요.

## 정성 분석 첨부 (`--notes`)

작업 기록·원인 분석을 마크다운으로 작성해 `--notes` 로 넘기면 리포트 하단에 **정성 분석 섹션**으로 붙습니다.
(제목·목록·표·코드블록·인용구 지원)

## 산출물

- 테마 대응(라이트/다크), 반응형, **의존성 없는 self-contained HTML** (내장 SVG 차트)

## 참고

- 이 저장소에는 **합성 샘플 데이터**만 포함됩니다. 실제 운영 데이터·자격증명·내부 IP 등 **민감정보는 커밋하지 마세요.**
