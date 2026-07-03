# Active Context — kordoc 본체

**마지막 업데이트**: 2026-07-03 (연속 세션 5: v3.8.3 릴리스 + formats 미달 3건 수술 + A-1 PDF 표 구조 GT 신설)
**상태**: 테스트 637/637. `npm run bench:gate` 4체인(score·roundtrip·formats·fuzz) 전부 PASS. tsc 13(동수)

## 이번 세션 완료 (2026-07-03 연속 5차)

- **⓪ v3.8.3 릴리스** — 2단 조판 읽기·한셀 XLSX·HML 표 복구 + 퍼즈 DoS 가드 묶음.
  npm publish + git tag + GitHub 릴리스(780ec49). CHANGELOG/README 현행화 동반
- **① docx 병합표 그리드 배치 재작성** (niied 0.675→**1.0**) — parseTable이 앵커 셀
  밀집 배열을 IRTable.cells(그리드 인덱스 규약)로 그대로 넘겨 렌더러(tableToHtml의
  skip-walk)가 gridSpan 뒤 셀을 통째 유실. **IR은 무죄, 렌더 규약 위반이 원인** —
  공용 buildTable(colAddr/rowAddr 직접 배치)로 교체. vMerge도 배열 인덱스→그리드 열
  기준으로 수정 (val 없는 `<w:vMerge/>`=계속 셀인데 기존 코드는 일반 셀로 오독해
  세로 병합이 아예 동작 안 했음). 셀 안 중첩표 텍스트 평탄화(hml 규약)로 arko
  0.880→0.9998
- **② docx 텍스트박스 수집** (kats 0.917→**0.9985**) — w:txbxContent를 아예 안 읽어
  KS표준안 422개 텍스트박스 전체 유실. collectTextboxParagraphs 신설: 앵커 문단 뒤
  별도 블록, **mc:Fallback 서브트리는 Choice 이중 렌더라 스킵**. 본문 워커+셀 수집
  양쪽 연결. 회귀 테스트 2건(병합표 그리드·텍스트박스 1회 출력) → 테스트 637
- **③ goe xlsx "다중시트 누락"의 진범 = 추출기** (str 0.984→**1.0**) — 파서 무죄.
  formats-sweep이 시트를 `sort()` 사전순(sheet10<sheet2)으로 걸어 md(워크북 순서)와
  유닛 순서가 어긋남 → align(순서 민감) 거짓 miss. workbook.xml+rels 순서 미러로
  수정. 추출기 docx 쪽도 Fallback 스킵+텍스트박스 별도 유닛으로 파서 미러
- **④ formats 플로어 상향** (2회 연속 동일 확인): docx 0.92→**0.998**, xlsxStr
  0.985→**0.999** (hml 0.995 유지 — bizinfo 글상자 잔존)
- **⑤ A-1 PDF 표 구조 GT 신설** (`bench/pdf-table-gt.mjs`) — pairs 6쌍에서 hwpx IR
  표(611/611 검증)를 참조로 pdf IR 표 scoreTables 대조. 비교 모수 = 최상위 2×2+
  (1×1 래퍼는 중첩표 승격, N×1 스트립 제외 — 글상자 관행으로 표 의미가 갈림).
  **기준선(2회 동일): ref 72표 매칭 0.8194 · exact 0.5139 · cellF1 0.6046 ·
  contentNED 0.4915**. 진짜 표 대부분 exact(pair11 12/18) — 잔여는 병합 열 표현 차·
  미감지 오매칭 연쇄·과분할. 게이트 편입은 개선 후

## 지표 대시보드 (2026-07-03 연속 5차 종료)

| 트랙 | 지표 | 값 | 게이트 | 비고 |
|---|---|---|---|---|
| hwpx(85) | recallMicro / phantom | **1.0** / 0.000054 | 0.999 / 0.005 | |
| hwpx | 표 exact / cellF1 | **611/611** / 1.0 | 0.99 / 0.999 | |
| hwpx | cellExact / contentNED | **1.0** / **1.0** | 0.999 / 0.9995 | |
| hwpx | orderAvg / eq / fn | 1.0 / 1 / 1 | 0.995/0.99/0.999 | |
| pdf(48) | coverage(micro) | **0.99609** | 0.985 | 미달 1건 = eval-perf-2024 0.9785(벡터 아웃라인) |
| hwp쌍(10) | 유사도 / 커버 | **0.9946 / 0.9929** | 0.99 / 0.99 | |
| docx(7) | recall | **0.998903** ⬆ | **0.998** ⬆ | niied·kats·arko 수술 완료, 잔여 kats 소량 |
| xlsx(11) | strRecall / numRecall | **1.0** ⬆ / 0.9844 | **0.999** ⬆(str) | num은 표기차 정상(보고만) |
| hml(9) | recall | 0.9960 | 0.995 | bizinfo 0.973(글상자 추정) |
| roundtrip | fwd / bwd / tblExact | 0.9473 / 0.9468 / 0.728 | 무후퇴 플로어 | 개선 백로그 4종 (헤딩 왕복이 1순위) |
| pdf표GT(6쌍) | 매칭/exact/cellF1/NED | 0.8194/0.5139/0.6046/0.4915 ✨신설 | 기준선만 | bench/pdf-table-gt.mjs, 게이트 편입은 개선 후 |
| fuzz(732런) | crash/hang/noCode/slow | **0/0/0/0** | 전부 0 | |
| 테스트/tsc | **637/637** / 13(동수) | — | — | +docx 회귀 2건 |

## 릴리스

- **v3.8.3 발행됨** (2026-07-03, 780ec49): 2단 조판 + DoS 가드 + 한셀/HML
- **v3.8.4 후보 적립 중**: docx 병합표+텍스트박스(1255e49 — 체감 버그픽스) — 다음 세션
  roundtrip/pdf 개선과 묶어 판단

## 다음 세션 (플랜: .claude/plans/next-session-roundtrip.md — 구 formats-surgery 대체)

- **① roundtrip 헤딩 왕복 소실 수술** (generator outline 스타일 미설정 추정 → tblExact
  0.728 연관 측정) → 플로어 상향
- **② pdf-table-gt 끌어올리기**: 병합 열 표현 차·과분할 조사 (시작점은 스크립트 헤더 주석)
- ③ 소액: hml bizinfo 글상자 / eval-perf-2024 OCR / A-5 폼 정오 / hwp3 합성 픽스처

## 재론 금지 (기존 결정 유지 + 신규)

- LINE_SEG 원본 유지 / 공문서 장평 95%·굴림체 1.0em·함초롬 0.97em / 한컴 빈 문단 생략형
- PDF 머리글/바닥글 y-클러스터 규칙 재도입 금지 (본문 오삭제 사고)
- PDF coverage 참조 trigram **줄 단위(perLine)** — 줄 경계 gram 재도입 금지
- hidden text 필터 회전 예외 유지 / **extractLines CTM 추적 제거 금지** / **pdfjs cMap 자산 지정 유지**
- **findTwoColumnProseCutX는 fullPage 호출에만** + **finite 가드·후보 상한 400 제거 금지** (fuzz DoS)
- align Pass 1 "본문문자 유닛 우선" 유지 — 마스킹-only 유닛이 본문 구간을 가로채는 함정
- **셀 장식 관용은 heading paraPr 마킹 줄에만** — 무조건 접두 제거로 완화 금지
- changwon 성능 재론 금지 — pdfjs 이미지 파이프라인 플로어 실측 완료 (kordoc 4.8%)
- **formats 추출기는 파서 경계 미러**: P 유닛에 중첩표 CHAR 쓸어담기 금지(hml 0.57 사건) /
  docx mc:Fallback 스킵+텍스트박스 별도 유닛 / **xlsx 시트 순서는 workbook 순서**
  (사전순 sort()가 goe 거짓 miss 사건)
- **pdf-table-gt 비교 모수 = 최상위 2×2+** (1×1 래퍼 중첩 승격) — N×1 스트립 재포함 금지
- docx vMerge: val 없는 `<w:vMerge/>`=계속 셀 (OOXML 기본값) — restart 아님

## 코퍼스/도구 메모

- 실파일 코퍼스(gitignore): review/ 45 · hwp5/ 13+30 · pdf/ 42 · pairs/ 26 · formats/ 27
- pairs 재수집: bench/pairs-manifest.json (mods.go.kr Referer 필수). formats 출처는
  수집 에이전트 로그 — 필요시 검색엔진 filetype 연산자로 재수집
- PDF 수집법: 검색엔진 `filetype:pdf site:go.kr` 직링크 (korea.kr RSS 폐지)
- score pdf 트랙 pdftotext(poppler) 필수 (/opt/homebrew/bin/pdftotext)
- 게이트 일괄: `npm run bench:gate` / perf: `node bench/perf.mjs` / PDF표: `node bench/pdf-table-gt.mjs`
- npm publish: `~/.npmrc` bypass 2FA granular 토큰 유효. 릴리스 관례 = release 커밋
  (CHANGELOG+README `## vX.Y.Z 변경사항`+package.json) + 경량 태그 + npm publish + gh release
