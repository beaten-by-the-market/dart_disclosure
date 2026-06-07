# DART 공시 수집 — OpenDartReader 미사용 프로젝트

[OpenDartReader 사용 프로젝트 인벤토리](OpenDartReader_프로젝트_정리.md)의 보완 스캔.
**OpenDartReader 패키지를 쓰지 않으면서** DART/거래소 공시 데이터를 수집하는 코드를 별도로 정리한다.

- 스캔 일자: **2026-06-08**
- 범위: github 26개 폴더 전체 (`dart_fss` / `opendart` / `dart.fss.or.kr` / `kind.krx.co.kr` / `crtfc_key` 등 패턴)
- 핵심 발견: **공식 DART Open API(`crtfc_key` + `/api/...` JSON)를 쓰는 코드는 워크스페이스에 없다.** 대신 ① DART 웹페이지 HTML 직접 크롤링, ② KIND(거래소 공시포털) 크롤링, ③ DART 기업개황 엑셀 다운로드 세 가지 방식을 쓴다.

---

## 수집 방식 분류

| 방식 | 설명 | 파일 수 |
|------|------|---------|
| **(C) DART 웹 HTML 크롤링** | `opendart.fss.or.kr/disclosureinfo/mainMatter/list.do`에 `requests.post` → `pd.read_html`/BeautifulSoup 파싱. 공식 API 아님 | 10 |
| **(D) KIND 거래소 공시포털 크롤링** | `kind.krx.co.kr` Selenium/`requests` 크롤링 | 3 |
| **(엑셀) DART 기업개황 다운로드** | `dart.fss.or.kr/dsae001/downloadExcel.do`로 상장사 마스터 엑셀 | 4 |
| **dart_fss / 공식 Open API** | — | **0 (없음)** |

> 같은 노트북이 여러 방식을 섞어 쓰는 경우가 많다(예: DART HTML + KRX OTP CSV + FinanceDataReader 주가).

---

## (C) DART 웹 HTML 크롤링 — `dart_disclosure/` 자사주·무상증자 노트북군

모두 `opendart.fss.or.kr/disclosureinfo/mainMatter/list.do`(주요사항보고서 통합검색 HTML)에 `reportCode`로 공시유형을 지정해 POST → HTML 테이블 파싱. **공식 Open API가 아니라 DART 웹 화면을 긁는 방식.** (이 노트북들은 KRX 인벤토리에도 "DART+KRX 결합 노트북"으로 잡혀 있다 — data.krx.co.kr도 함께 호출하기 때문.)

| 파일 | reportCode | 수집 대상 | 함께 쓰는 소스 |
|------|-----------|-----------|----------------|
| `kospi_dart_buyback.ipynb` | 11332 | KOSPI 자사주 직접취득 예정 수량·금액 (2015~2024) | — |
| `kospi_dart_buyback_resale.ipynb` | 11332 | KOSPI 자사주 취득/처분 (금감원 vs 거래소 신고 비교) | KIND 크롤 |
| `kospi_dart_bonusissue.ipynb` | 11307 | KOSPI 무상증자 결정·발행비율 분포 | — |
| `kosdaq_buyback.ipynb` | 11332 | KOSDAQ 자사주 취득·장내 누계 | KRX OTP CSV |
| `kospi_buyback.ipynb` | 11332 | KOSPI 자사주 취득 예정 vs 장내 체결 | KRX OTP CSV |
| `buyback_period.ipynb` | 11332 | 자사주 취득기간 중 주가 추이 | FinanceDataReader |
| `buyback_resale_blockdeal.ipynb` | 11332/11333 | 자사주 처분(시간외대량매매) vs 주가 | FinanceDataReader |
| `buyback_blockdeal.ipynb` | 11332 | 자사주 시간외대량매매 사례 (2015~2024) | — |
| `bonusissue_getprice.ipynb` | (무상증자) | 무상증자 후 주가 변화 | DART 기업개황 엑셀 + KRX OTP |
| `자사주_소각KIND.ipynb` | (소각) | 자사주 소각 내역 | KIND 크롤 + 기업개황 엑셀 |

> 블로그 리포(`beaten-by-the-market.github.io/_posts/`)의 다수 글(kospi_dart_buyback, bonusissue, buyback_blockdeal 등)이 위 노트북 코드를 그대로 옮긴 예제다 — 데이터 수집 프로젝트가 아니라 발행물이므로 별도 카운트하지 않음.

---

## (D) KIND 거래소 공시포털 크롤링 — `fairdisclosure/`

`kind.krx.co.kr` 상세검색 화면을 Selenium/`requests`로 조작해 테이블을 긁는 방식. DART가 아니라 거래소(KRX) 공시포털이지만 "공시 수집"이라 함께 정리.

| 파일 | 수집 대상 |
|------|-----------|
| `fairdisclosure/earningschangedisc.py` | 손익구조변경 의무공시 (2014~2023, KIND Selenium) |
| `fairdisclosure/fairdisc_kind.py` | 공정공시(fair disclosure) (2014~2023, KIND) |
| `dart_disclosure/자사주_소각KIND.ipynb` | (위 C 항목과 중복 — KIND 변경상장도 크롤) |

---

## (엑셀) DART 기업개황 다운로드 — 마스터 데이터 보조

`dart.fss.or.kr/dsae001/downloadExcel.do`로 시장별 상장사 기업개황 엑셀을 받아 법인 마스터를 만드는 용도. 단독 공시 수집이라기보다 다른 파이프라인의 종목 매핑 보조.

| 파일 | 용도 |
|------|------|
| `seibro-api/seibro_api/corp_loader.py` | DART 기업개황 엑셀 + SEIBRO 예탁원 고객번호 → 법인 마스터 |
| `seibro/seibro_data_get_azure.py` | 기업개황 엑셀 + SEIBRO OpenAPI 수집 |
| `dart_disclosure/bonusissue_getprice.ipynb` | (위 C 중복) |
| `dart_disclosure/자사주_소각KIND.ipynb` | (위 C 중복) |

---

## 참고: 공시 수집이 아닌 것 (오탐 제외)

스캔에 걸렸으나 DART 데이터를 **새로 수집하지 않는** 파일 — 대부분 이미 MySQL에 적재된 데이터를 읽어 분석:

- `fairdisclosure/fairdisc_analysis.py`, `fairdisc_analysis2.py` — KIND 적재분(공정공시·손익변경) MySQL 분석
- `ad_hoc_issues/categorize.py` — MySQL `disc_list` 배당결정공시 NLP 분류
- `krxnewsscrap/bonus_issue.py` — MySQL `st_bonus_issue` 무상증자 Streamlit 시각화
- `seibro/seibro_anlysis.py`, `seibro-api/seibro_api/display.py` — SEIBRO 데이터 분석/표출
- `edgar_8_k/8k_301.py` — **SEC EDGAR(미국)** 8-K Item 3.01, DART 아님
- `beaten-by-the-market.github.io/_posts/*.md`, `*.html` — 블로그 발행물/출력물

---

## 종합

- **OpenDartReader 사용**: 9개 폴더 / 27개 파일 → [OpenDartReader 프로젝트 정리](OpenDartReader_프로젝트_정리.md)
- **OpenDartReader 미사용 DART/공시 수집**: 주로 `dart_disclosure/`의 자사주·무상증자 노트북 10개(DART 웹 HTML 크롤) + `fairdisclosure/`의 KIND 크롤 2개 + 기업개황 엑셀 보조 2개.
- **공식 DART Open API(`crtfc_key`) / `dart_fss` 라이브러리는 워크스페이스 어디에도 사용되지 않음.**
- 통합 여지: OpenDartReader 경로와 별개로, `opendart.fss.or.kr/disclosureinfo` HTML 크롤 패턴(reportCode 지정 + read_html)도 자사주·무상증자 노트북 전반에 중복되어 있어 별도 헬퍼로 추출 가능.
