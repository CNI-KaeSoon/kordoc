# Active Context — kordoc 본체

**마지막 업데이트**: 2026-07-02 (v3.8.0 릴리스 세션 — HWPX 이미지 dedupe·docx 관측성·릴리스 완료)
**상태**: 테스트 621/621. **v3.8.0 npm 게시 + origin/main 푸시 완료** (release 커밋 `7850c6c`)

## 이번 세션 완료 (2026-07-02 저녁)

- **A. HWPX 이미지 ref 단위 dedupe** (`a13fff3`) — `hwpx/parser.ts` extractImagesFromZip:
  ref당 1회 해제·변환 캐시 + 데이터 버퍼 공유, 실패도 캐시해 SKIPPED_IMAGE 경고 1회.
  ZIP bomb 가드(MAX_DECOMPRESS_SIZE)는 실해제만 가산(캐시 히트 미가산 — 의미 보존 확인).
  HWP5 픽스(18fe5d9) 미러. 검증: perf 회귀 없음(hwpx median 7.66ms·no-op 88/88·big_file 186ms),
  score.mjs 전게이트 PASS, 테스트 +2 (같은 ref 3회 참조 버퍼 공유 / 실패 캐시 경고 1회)
- **B. docx 무경고 catch 5개 경고화** (`eabff7b`) — 이미지=SKIPPED_IMAGE,
  스타일/번호/각주/메타=PARTIAL_PARSE. extractImages에 warnings 인자 추가.
  파싱은 계속(동작 동일). 깨진 styles.xml 픽스처 테스트 +1 (xmldom fatal throw 확인)
- **C. v3.8.0 릴리스** (`7850c6c`) — CHANGELOG·README(v3.8.0 변경사항 섹션+라운드트립 문구) 현행화,
  npm publish(~/.npmrc bypass 2FA 토큰, prepublishOnly가 test+build 실행), registry 3.8.0 확인, push 완료

## D 보류 — pdf/parser.ts 2,417줄 분리 (선행 조건 미충족)

- ⚠️ **korea.kr RSS 전면 404** (pressrelease/policy/briefing/news 등 전부 + /rss/ 인덱스까지)
  — `bench/collect-korea-kr.mjs` 무력화 (2026-07-02 확인)
- 보도자료 뷰(briefing/pressReleaseView.do)는 200이나 **첨부파일 없음** (본문 임베드형 개편 추정)
- 부처 직접 수집도 정찰 실패: 행안부(mois) 목록 정적 HTML에 nttId 없음(JS 렌더), 기재부(moef) 301
- **다음 방향**: 부처 보도자료(hwpx+pdf 병행 첨부 관행) 크롤러 신규 작성 또는 다른 공공 PDF 소스 발굴
  → 실PDF 20건+ 확보 → score.mjs pdf_cross_coverage 기준선 → 순수 이동 분리(전/후 markdown 해시 동일 스윕)
- 분리 대상 6책임: 메타추출 / XY-Cut 읽기순서 / 헤딩감지 / 머리말·바닥글 제거 / 표감지 위임 / 수식삽입
- 그다음 후보: hwpx/parser.ts(1,690줄대)·pdf/line-detector.ts(1,247줄)

## 기준선 (v3.8.0, 2026-07-02)

- 테스트 **621/621** (`npm run build` 후 실행 — dist 스테일 함정)
- score.mjs review 45건: recallMicro 0.999911·phantom 0.000112·표 217/217 전게이트 PASS
  (기존 결손 1건 유지: review/36434527 "가." 2자 — 백로그)
- perf.mjs: hwpx median 7.7~7.8ms·11.7MB/s / big_file 186ms / no-op 88/88 바이트동일
- verify-linebreak: 굴림체 56/57 (98%)

## 남은 백로그 (전부 저순위)

- 표 열/병합 변경, 1x1·1열 표 행 연산, HWP5 행 추가/삭제 (P1 스코프 제외분)
- HWP5 중첩표 셀 수정 — ScanCell5에 tables 없음, 스캔 구조부터 필요 (최후순위)
- IR filler 전략2: 데이터 행 병합 시 hwpx tc 서수 페어링 열 어긋남 가능 (기록만)
- GFM 경로 다문단 빈 셀(빈 문단 2+)은 "셀 문단 수 변경" skip (정직 보고 — HTML 경로는 지원)
- 본문(비셀) 빈 문단 채우기 — 마크다운에 유닛이 없어 설계상 불가 (기록만)
- score.mjs recall 결손 1건: review/36434527 "가." 2자 누락 — 원인 조사 후보

## 재론 금지 (기존 결정 유지)

- LINE_SEG 원본 유지 (한컴이 재배치, 단일화하면 글자 겹침 — 실측), lineseg 제거는 수정 섹션만
- 공문서 전역 장평 95%, 굴림체=1.0em 고정폭, 함초롬 한글=0.97em, 공백=0.5em
- 한컴 빈 문단 = PARA_TEXT 생략형(nChars=1) 지배적 — 재실측 불필요
- 정보소통광장은 구형 문서도 hwpx 변환 제공 — .hwp 수집 불가, hwplib 픽스처 사용
- session changes vs patchHwpx verification 의미 차이 — 의도됨
- 빈 문자열 블록 비우기 = skip (블록 핸들 소실 방지)

## 코퍼스/도구 메모

- 실파일 코퍼스(gitignore): `bench/corpus/review/` hwpx 45건, `bench/corpus/hwp5/` .hwp 13건(hwplib 12 + big_file 10MB) + hwpx 30건(2016). PDF 0건
- 실.hwp 픽스처 = hwplib(neolord0) sample_hwp 12건. hwp5-patch.test.ts 실파일 스윕은 코퍼스 없으면 skip
- npm publish: `~/.npmrc` bypass 2FA granular 토큰으로 비대화형 성공 (2026-07-02에도 유효 확인)
