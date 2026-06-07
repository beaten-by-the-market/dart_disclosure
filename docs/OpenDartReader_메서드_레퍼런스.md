# OpenDartReader 메서드 사용 레퍼런스

본 워크스페이스 27개 파일에서 실제 호출되는 `OpenDartReader` 인스턴스 메서드/속성과 사용처 정리.
전체 인벤토리는 [OpenDartReader 프로젝트 정리](OpenDartReader_프로젝트_정리.md) 참조. (스캔 2026-06-08)

## 호출 빈도

| 메서드/속성 | 파일 수 | 역할 |
|-------------|---------|------|
| `dart.list()` | 18 | 기간·공시유형으로 공시 목록(접수번호) 조회 — 거의 모든 흐름의 출발점 |
| `dart.sub_docs()` | 8 | 접수번호의 첨부 하위문서 목록/URL — 본문 HTML 파싱용 |
| `dart.event()` | 4 | 종목별 특정 이벤트 공시 이력(무상증자·유상증자·CB/BW/EB·자기주식처분 등) |
| `dart.document()` | 3 | 접수번호의 공시원문 XML 다운로드 |
| `dart.finstate_all()` | 2 | 정기보고서 XBRL 전체 재무제표 |
| `dart.report()` | 1 | 사업보고서 인사·재무 단일항목(보수 등) |
| `dart.company()` | 1 | 종목코드 기반 기업 개황 |
| `dart.corp_codes` | 1 | 전 상장사 고유번호↔회사명 매핑 (속성) |

## 메서드별 사용처

### `dart.list(corp, start, end, kind, kind_detail, final)`
공시 목록 조회. 가장 광범위하게 사용.
- 공시유형 인자: `kind='I'`(거래소 수시), `kind=['B','I']`, `kind_detail='A001'`(사업보고서), `'A003'`(분기), `'B001'`(주요사항), `'E004'`(주식매수선택권), `'E006'`(주총소집공고)
- 사용: snowflake_dart_annualrep, disclosure_terms, dart_xml_example, DART_API실습, dart_report.py, dart_stockoption_extract, stockoption_endow, dart_gcd_extract, get_xbrl_full_opendartreader(_for_real), eng_discl_support.v2, supply_contract, notice_disc, stock_option, forecast_real, 사업보고서_미상환CBBW, 주주총회소집공고_정관변경건, get_disc_upload_azure

### `dart.sub_docs(rcept_no)`
접수번호의 하위 첨부문서 목록(제목·URL). URL을 받아 BeautifulSoup로 본문 파싱하는 패턴.
- 사용: supply_contract, notice_disc(자체 구현 변형), stock_option, forecast_real, 사업보고서_미상환CBBW, 주주총회소집공고_정관변경건, get_disc_upload_azure, dart_ir_analysis

### `dart.event(stock_code, event_type, start, end)`
종목+이벤트 유형 공시 이력.
- 이벤트 인자: `'무상증자'`, `'유상증자'`, `'자기주식처분'`, CB/BW/EB 발행
- 사용: get_bonus_issue(무상증자), DART_API실습(유상증자·자기주식처분), stock_bond(CB/BW/EB), elec_cb_status(CB)

### `dart.document(rcept_no)`
공시원문 XML 다운로드 → XML 파싱.
- 사용: dart_xml_example, dart_report.py, sec_reg_statement

### `dart.finstate_all(corp, bsns_year, reprt_code, fs_div)`
정기보고서 XBRL 전체 재무제표.
- 사용: get_xbrl_full_opendartreader, get_xbrl_full_opendartreader_for_real

### `dart.report(corp, key_word, bsns_year)`
사업보고서 단일항목(임원·개인별 보수 등).
- 사용: annual_report_items

### `dart.company(corp)`
기업 개황(기업명·산업코드 등).
- 사용: get_corp_status

### `dart.corp_codes` (속성)
전 상장사 고유번호↔종목코드↔회사명 매핑 DataFrame.
- 사용: report_validation_xbrl_opendartreader
- 관련 캐시: `docs_cache\opendartreader_corp_codes_20241116.pkl` (현재 코드 참조 없음 — 프로젝트 정리 문서의 "캐시 아티팩트 주의" 참조)

## 전형적 흐름

```python
dart = OpenDartReader(API_KEY)

# 1) 목록
docs = dart.list('20240101', '20241231', kind_detail='A001', final=False)

# 2-A) 원문 XML
xml = dart.document(rcept_no)            # → ElementTree 파싱

# 2-B) 첨부 본문 HTML
subs = dart.sub_docs(rcept_no)           # → URL → requests/BeautifulSoup 파싱

# 이벤트 직접 조회 (목록 단계 생략)
ev = dart.event(stock_code, '무상증자', start='20220101', end='20231231')
```
