# Active Context — kordoc 본체

**마지막 업데이트**: 2026-07-02 (v3.6.0 프로덕션 리뷰 — 커밋 완료)
**상태**: v3.6.0 = main `50bf9bf` 커밋됨(푸시 안 함), 테스트 577/577. **npm publish 미실행**(registry는 아직 3.0.1).
**다음 세션**: `.claude/plans/next-session-gaps.md` 프롬프트로 잔여 갭(P1 표 구조 변경부터) 착수 — 렌더 하네스는 `.claude/plans/render-rhwp.mjs`

## v3.6.0 이번 세션 요약 (2026-07-02)

### 신규 핵심: 텍스트 메트릭 엔진 (src/hwpx/text-metrics.ts)
- 함초롬바탕 정품 TTF advance 전수 추출: 한글 11,172자 균일 970/1000em, 숫자 550, 온점·괄호 320, **Bold=Regular 폭 동일**
- 줄바꿈 시뮬레이션: keep(어절)/charAll(글자) + 금칙(시작=직전 1글자 동반 밀어내기, 끝=여는괄호 내리기)
- **오라클 검증**: bench/verify-linebreak.mjs — 실제 결재문서 linesegarray 대조, 고정폭 버킷 98%(56/57)
- 확정 규칙: 공백 0.5em 고정(useFontSpace=0), 장평·자간 공백에도 적용, 자간=폭×(1+sp/100)
- ⚠️ 전자결재 변환기·macOS한컴·rhwp는 KEEP_WORD 무시하고 글자단위 조판 (Windows 한컴만 어절)

### 수정 (프로덕션 버그)
1. **fillHwpx/HwpxSession linesegarray 미제거** → 한컴 변조경고/줄배치 틀어짐. patcher처럼 수정 섹션 lineseg 전부 제거
2. **생성 표 테두리 안 보임** — borderFill id 0-시작이 원인. **1-based 규약**(1=무테두리, 2=SOLID) + centerLine="NONE"(enum). 실전 파일에 id=0 없음
3. markerWidth 실측화 (괄호 0.45→0.32em 등) — 내어쓰기 정렬 오차 제거

### 신규 기능
- **autoFit(문단별 자동 장평)**: orphan 문단만 95→90 축소, 변형 charPr(id 11+) 발급. GongmunOptions.autoFit
- **HTML 표(colspan/rowspan/중첩) → HWPX 생성**: generateHtmlTableXml — parse↔generate 표 라운드트립 완성
- **다중값 채우기**: values에 string[] — 반복 라벨 순서 소진 + 명부형 표 행별 채움 (ValueCursor, match.ts)

### 도구/인프라
- bench/collect-opengov.mjs 사이트 개편 대응 재작성 (title-down에서 파일명, 제목 필터 인자)
- bench/corpus/review/ 실문서 45건 (gitignore) — e2e DIRS에 "review" 추가로 실파일 스윕 활성화
- 검증 하네스(로컬 스크래치): @rhwp/core 렌더 → Chrome headless SVG→PNG로 육안 확인 가능

### 남은 백로그 (이번 리뷰에서 확인된 갭 — 미착수)
- 표 구조 변경(행/열 추가·삭제, 병합 변경) 전 경로 미지원 — 최상위 갭
- HWP5: 중첩표 셀 수정·빈 셀 채우기·빈 문단 삽입·문단→표 미지원 (HWPX만 지원)
- IR filler: 병합 라벨셀 값 유실(silent), 중첩표 라벨 미재귀 — hwpx-preserve 경로와 불일치
- 라벨 인식: 숫자 낀 라벨(연번1 등)·9자↑ 한글·콜론 없는 영문 미인식
- 셀 줄 병합 시 applied+skip 이중 보고 (silent 손실 가능)
- buildTableWithCellMeta 재부착 실패 시 중첩표 blocks 유실 (hwpx/parser.ts:864)
- 수식+병합/중첩 → GFM 강등으로 구조 소실 (builder.ts:430)
- HWP5 patchHwp 전용 테스트 부재
- 리스트 사이 표가 끼면 공문서 번호 run이 끊겨 재시작 (마크다운 표현 한계)

### 함정 메모
- verify-linebreak 오라클: textpos가 텍스트 길이 초과하는 파일 있음(원본 hwp 컨트롤 좌표 승계) — coordShift로 제외
- 굴림체=고정폭 1.0em 확정 (970 아님). 한컴돋움은 비례폭이라 HCR 근사 부정확
- dist 스테일 함정 여전 — src 수정 후 npm run build 필수
