# OpenDartReader 사용 프로젝트 인벤토리

`c:\Users\Peter\github` 작업 디렉터리 하위에서 **OpenDartReader** 패키지(`opendartreader`)로 DART 전자공시 데이터를 수집·파싱하는 프로젝트 정리.

- 스캔 일자: **2026-06-08**
- 범위: `OpenDartReader` import 또는 `dart.*()` 호출이 있는 `.py` / `.ipynb`
- 규모: **9개 폴더 / 27개 파일**
- 자매 문서: KRX 정보데이터시스템 호출 인벤토리(`krx-data-api\docs\KRX_데이터_프로젝트_정리.md`), 본 폴더의 [OpenDartReader 메서드 레퍼런스](OpenDartReader_메서드_레퍼런스.md)

> OpenDartReader는 DART Open API(고유번호·공시목록·정기보고서 항목)와 공시뷰어 원문(XML/HTML)을 한 객체로 묶어주는 래퍼다. 거의 모든 프로젝트가 `dart = OpenDartReader(api_key)` 한 줄로 시작해 `dart.list()`로 공시 목록을 받고, 필요 시 `dart.document()` / `dart.sub_docs()`로 본문을 내려받아 파싱한다.

---

## 폴더 목록 (파일 수 순)

| 폴더 | 파일 | 주된 용도 | 결합 소스 |
|------|------|-----------|-----------|
| `xbrl_validation/` | 6 | XBRL 재무·스톡옵션·감사 정보 추출/검증 | MySQL, KRX |
| `ad_hoc_issues/` | 5 | 수시공시(공급계약·무상증자·관리종목·기타안내) 분석 | KRX, MySQL |
| `dart_disclosure/` | 4 | 사업보고서·당일공시 대량수집, XML/이벤트 실습 | Snowflake, OpenAI, KRX |
| `seibro-api/` | 3 | 주식관련사채(CB/BW/EB) 공시·정기보고서 교차검증 | SEIBRO |
| `adhoc_shareholdermeeting_agenda/` | 2 | 사업보고서 미상환 CB/BW, 주총 정관변경 안건 | KRX, SEIBRO |
| `forecast_real/` | 2 | 특례상장 실적예측 괴리·임원 스톡옵션 | KRX, KIND |
| `sec_reg/` | 2 | 신규상장 공모정보·기업정보 매핑 | — |
| `fairdisclosure/` | 2 | 사업보고서 자금조달·IR 공시 본문 수집 | KRX, MySQL |
| `seibro/` | 1 | CB 발행회사 전자등록 여부 조사 | KRX, SEIBRO |

---

## 폴더별 상세

### `xbrl_validation/` — XBRL 재무/공시 추출·검증 (6)
- **`get_xbrl_full_opendartreader.py`** — `dart.list(kind_detail='A003')` + `dart.finstate_all()`. 3분기보고서 XBRL 재무제표 8개 핵심계정(자산·부채·자본·매출·영업이익·법차손·순이익) 추출 → MySQL.
- **`get_xbrl_full_opendartreader_for_real.py`** — 위와 동일 구조, 1분기보고서 버전.
- **`dart_stockoption_extract.py`** — `dart.list(kind='A001')`. 2023 사업보고서에서 주식매수선택권 부여현황 XML 파싱. (KRX 종가·상장주식수·시총 결합)
- **`stockoption_endow.py`** — `dart.list(kind_detail='E004')`. 2021~2024 주식매수선택권 부여신고서 공시 이력 수집.
- **`dart_gcd_extract.py`** — `dart.list(kind='A001')`. 2023 사업보고서 감사정보(감사인·감사의견·감사유형)를 XBRL GCD 택사노미로 추출.
- **`report_validation_xbrl_opendartreader.py`** — `dart.corp_codes`. 고유번호↔종목코드 매핑으로 XBRL(BS/IS/SCE) 재무 검증. (MySQL 적재 데이터 기준)

### `ad_hoc_issues/` — 수시공시 분석 (5)
- **`supply_contract.py`** — `dart.list(kind='I')` + `dart.sub_docs()`. 단일판매·공급계약 공시에서 금액·비율·기간·유보사유 추출. (KRX listed_stocks)
- **`annual_report_items.py`** — `dart.report(item, year)`. 2015~2023 사업보고서 인사항목(개인별·임원 보수) 수집.
- **`get_bonus_issue.py`** — `dart.event('무상증자')`. 무상증자 결정 내역 + 전후 주가 수익률. (KRX 가격, MySQL)
- **`eng_discl_support.v2.py`** — `dart.list(kind='I')`. 자산 3천억 미만 상장사의 관리종목·횡령배임·실질심사 정지 이력. (KRX supervised/unfaithful)
- **`notice_disc.py`** — `dart.list(kind='I')` + 자체 `sub_docs()`. 기타시장안내(Notice) 공시 본문 추출(종목 014200).

### `dart_disclosure/` — 대량수집·실습 (4)
- **`snowflake_dart_annualrep.ipynb`** — `dart.list(kind_detail='A001')`. 2020~2025 사업보고서 ~20,051건 수집 → Snowflake. (첨부정정 원본 접수번호는 DART 웹 크롤링으로 추적)
- **`disclosure_terms.ipynb`** — `dart.list(kind=['B','I'])`. 당일 거래소 수시공시·주요사항보고서에서 용어 추출. (OpenAI gpt-4o-mini + LangChain)
- **`dart_xml_example.ipynb`** — `dart.list()` + `dart.document(rcept_no)`. 개별 공시 XML 원문 수집·파싱 실습.
- **`DART_API실습.ipynb`** — `dart.list(kind_detail='B001')` + `dart.event('유상증자')` / `dart.event('자기주식처분')`. 유가증권 기업 주요공시 분석(자금조달 목적·자사주 처분 목적 분류).

### `seibro-api/` — 주식관련사채 교차검증 (3)
- **`seibro_api/dart_report.py`** — `dart.list()` + `dart.document()`. 사업/반기보고서의 채무증권 발행실적(SUB_PIS)·CB·BW·EB 테이블 파싱.
- **`seibro_api/stock_bond.py`** — `dart.event()`. 단일 종목 CB/BW/EB 발행결정 공시 이력.
- **`bond_summary.ipynb`** — 위 패키지 함수(`get_dart_cb_events`, `get_bonds_from_report` 등) 호출. 예탁원·DART공시·정기보고서 3소스 교차검증.

### `adhoc_shareholdermeeting_agenda/` — 주총/사채 안건 (2)
- **`사업보고서_미상환CBBW.py`** — `dart.list(kind_detail='A001')` + `dart.sub_docs()`. 사업보고서 미상환 CB·BW 정보. (KRX, SEIBRO)
- **`주주총회소집공고_정관변경건.py`** — `dart.list(kind_detail='E006')` + `dart.sub_docs()`. 주총소집공고에서 정관변경 의안·변경내용 추출. (KRX)

### `forecast_real/` — 특례상장·스톡옵션 (2)
- **`forecast_real.py`** — `dart.list(kind_detail='A001')` + `dart.sub_docs()`. 특례상장 기업 사업보고서의 예측치 vs 실적치 괴리율. (KRX new_listing, KIND 크롤링)
- **`stock_option.py`** — `dart.list(kind_detail='A001')` + `dart.sub_docs()`. 사업보고서 임원 스톡옵션 부여·미행사 내역. (KRX)

### `sec_reg/` — 신규상장 공모 (2)
- **`sec_reg_statement.py`** — `dart.document()`. 신규상장 공모정보(청약·배정현황, 인수기관별 인수금액, 자금조달내용) 추출.
- **`get_corp_status.py`** — `dart.company()`. 종목코드 기반 기업정보(기업명·산업코드) 조회·매핑.

### `fairdisclosure/` — 자금조달·IR (2)
- **`get_disc_upload_azure.py`** — `dart.list(kind_detail='A001')` + `dart.sub_docs()`. 사업보고서 증권발행 자금조달 정보 수집·HTML 저장. (KRX)
- **`dart_ir_analysis.py`** — `dart.sub_docs()`. 거래소 공시목록(MySQL `disc_list`) 중 IR 공시 본문 HTML 추출(최근 5년).

### `seibro/` — CB 전자등록 조사 (1)
- **`elec_cb_status.py`** — `dart.event()`. CB 발행회사의 전자등록 여부 조사. (KRX 표준코드, SEIBRO 주식관련사채)

---

## 캐시 아티팩트 주의

`dart_disclosure\docs_cache\opendartreader_corp_codes_20241116.pkl` (5.3MB, 2024-11-16 생성)
— `dart.corp_codes`(전 상장사 고유번호↔회사명 매핑)를 피클로 저장해 둔 캐시로 추정되나, **현재 워크스페이스 코드 어디에서도 이 경로를 읽거나 쓰는 곳을 찾지 못했다.** 과거 생성 후 참조가 끊긴 고아 파일일 가능성이 높다. corp_codes 캐싱을 다시 쓸 경우 생성·로드 코드를 새로 연결할 것.

---

## 공통 패턴 메모

- **초기화**: `dart = OpenDartReader(API_KEY)` — API 키는 파일별로 하드코딩되거나 환경변수로 주입. (KRX 패키지와 달리 별도 추출 패키지 없음)
- **목록→본문 2단계**: 대부분 `dart.list(...)`로 접수번호(rcept_no)를 얻고 → `dart.document()`(XML) 또는 `dart.sub_docs()`(첨부 하위문서 URL) → 직접 파싱(XML/BeautifulSoup).
- **공시유형 코드**: `A001` 사업보고서, `A003` 분기보고서, `B001` 주요사항보고서, `E004` 주식매수선택권부여, `E006` 주주총회소집공고, `kind='I'` 거래소 수시공시, `kind='B'` 정기.
- **결합 소스**: 14개 파일이 KRX(상장종목 마스터·가격)와 결합, SEIBRO·MySQL·Snowflake·OpenAI 등과도 연계.
- **통합 패키지 가능성**: KRX 호출을 `krx-data-api`로 묶었듯, 반복되는 `dart.list→sub_docs→파싱` 패턴도 별도 패키지로 추출할 여지가 있다.
