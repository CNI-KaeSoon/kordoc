# Active Context — kordoc 본체

**마지막 업데이트**: 2026-07-03 (연속 세션 2: PDF 루트픽스 2건 + generator 분리 → **v3.8.2 릴리스**)
**상태**: 테스트 628/628. score 전 게이트 PASS. **v3.8.2 npm 게시 + GitHub Release + main 동기화 완료**

## 이번 세션 완료 (2026-07-03 연속 2차)

- **①-a. PDF 선 추출 CTM 추적** (`fix:`) — 콘텐츠 스트림 변환([0.75,0,0,-0.75,0,H]) 하의
  괘선이 텍스트와 다른 좌표계에 놓여 그리드-텍스트 매핑 전멸 → 괘선 표가 클러스터 경로로
  떨어져 **2줄 셀이 행으로 분할**되던 다행 셀 문제의 루트 원인. extractImageRegions와 동일한
  save/restore/transform 추적 + lineWidth/얇은사각형 판별을 사용자 공간 기준으로.
  eval-perf-2024 헤더 rowspan=2 정상 병합. coverage 0.99471→0.99581 (8↑ 3↓미미, 항등 CTM 30건 바이트동일)
- **①-b. pdfjs CID 폰트 자산 지정** (`fix:`) — cMapUrl/standardFontDataUrl 미지정으로 CMap 필요
  폰트 텍스트가 무음 소실. nanet-seoul-minutes(66p 의사록): "스캔본" 오판정·빈 출력 →
  **129KB 전문 추출**(poppler도 못 읽음). bench 참조 pdfjs에도 동일 적용(대칭).
  coverage 0.99591, **42건 전건 정식 채점**(ocr격리·weak 0)
- **②. hwpx/generator.ts 1,068줄 → 7모듈** (`refactor:`) — 엔트리 63줄.
  gen-ids(166)/md-runs(212)/gen-header(208)/gen-gongmun-fit(114)/gen-table(178)/gen-section(204).
  **gen-sweep 게이트 신설**(bench/gen-sweep.mjs): zip 타임스탬프 비결정이라 내부 엔트리 sha256
  스냅샷(픽스처 5종 30엔트리) — 전/후 동일
- **v3.8.2 릴리스**: CHANGELOG·README·CLAUDE.md 모듈표 현행화, npm 게시(registry 확인), tag+GitHub Release

## 기준선 (2026-07-03 v3.8.2)

- 테스트 **628/628** (build 후 실행 — dist 스테일 함정. **tsup 타입체크 안 함** → 분리/이동 후 `npx tsc --noEmit`, 기존 에러 13건·신규 0 기준)
- score 전체 PASS: hwpx 전 게이트 / **pdf 42건 coverage 0.99591, 전건 채점**
- pdf per-doc 미달 2건(게이트는 micro): eval-perf-2024 0.9785(잔여=벡터 아웃라인 숫자 — OCR 영역, 텍스트 추출 불가), assembly-minutes-1179 0.9836(미분석)
- perf: hwpx median ~7.9ms / no-op 88/88
- 게이트 도구: `bench/hash-sweep.mjs`(파싱 방향) · `bench/gen-sweep.mjs`(생성 방향, 신설)
- 대형 파일 전부 해소: 최대 pdf/cluster-detector 727줄(관찰), hwpx/section-walker 659줄(상호재귀 클러스터 — 의도)

## 다음 후보 (우선순위순)

1. **assembly-minutes-1179 0.9836 분석** — 마지막 per-doc 미달 중 미분석 건
2. **score recall 결손 1건**: review/36434527 "가." 2자 누락 — 원인 조사
3. **회전 표 읽기 순서** — 회전 텍스트가 아이템 단위로만 포함됨(khs p770 계속비 표). 회전 그룹별 좌표 리맵 검토
4. 기능 백로그(표 열/병합 변경, 1x1·1열 표 행 연산, HWP5 행 추가/삭제) 중 선별

## 남은 백로그 (전부 저순위)

- PDF: 텍스트순서 mid 바닥글(cbe/ice-arc 잔여), mdToPlain ASCII '|' 비대칭(기록만), eval-perf 벡터 아웃라인 숫자(OCR 영역)
- HWP5 중첩표 셀 수정(스캔 구조 필요, 최후순위), IR filler 전략2 병합 행(기록만)
- GFM 다문단 빈 셀 skip(정직 보고), 본문 빈 문단(설계상 불가)

## 재론 금지 (기존 결정 유지)

- LINE_SEG 원본 유지 / 공문서 장평 95%·굴림체 1.0em·함초롬 0.97em / 한컴 빈 문단 생략형
- PDF 머리글/바닥글 y-클러스터 규칙 재도입 금지 (본문 오삭제 사고)
- PDF coverage 참조 trigram **줄 단위(perLine)** — 줄 경계 gram 재도입 금지. kordoc 측만 전체 텍스트
- hidden text 필터 회전 예외 유지 — 진짜 0 스케일([0,0,0,0])만 hidden
- **extractLines CTM 추적 제거 금지** — 변환 깔린 문서(성과계획서류)의 표 감지가 전멸함
- **pdfjs cMap 자산 지정 유지** — 없으면 CID 폰트 텍스트층 무음 소실(nanet 129KB 사례)

## 코퍼스/도구 메모

- 실파일 코퍼스(gitignore): review/ 45건 · hwp5/ 13+30건 · pdf/ 42건
- PDF 수집법: 검색엔진 `filetype:pdf site:go.kr` 직링크 (korea.kr RSS 폐지)
- score pdf 트랙 pdftotext(poppler) 필수 (/opt/homebrew/bin/pdftotext)
- npm publish: `~/.npmrc` bypass 2FA granular 토큰 유효 (3.8.2 게시로 재확인)
