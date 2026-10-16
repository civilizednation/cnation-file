# cnation-file (앱 이름: cnation File, 옛 이름 Clip Saver)

클립보드 내용을 원하는 파일명의 파일로 저장하는 개인용 웹앱. 이 레포의 작업 맥락을 정리한 메모입니다.

## 현재 상태
- 앱 전체가 루트의 `index.html` 한 파일(빌드 도구 없음). 처음엔 `cnation-ft`(게임 프로젝트) 레포의 `clip-saver/` 폴더에서 만들었다가 이 레포로 옮겨 PR #1로 `main`에 머지됨.
- 기능
  - 파일명 입력: 기본값 `index.html`, 마지막 파일명을 localStorage(`clipSaver.filename`)에 기억
  - 클립보드 붙여넣기 버튼: iPhone에선 "붙여넣기" 확인 말풍선이 뜸. 읽기 실패 시 입력칸을 길게 눌러 직접 붙여넣도록 안내
  - 저장 버튼: Web Share API로 공유 시트 → "파일에 저장". 공유가 안 되는 브라우저(PC 등)는 바로 다운로드
  - 확장자별 MIME(html, css, js, json, md 등), 다크모드 지원
- 클립보드 읽기와 공유는 HTTPS에서만 동작함(`file://` 불가). Vercel 또는 GitHub Pages로 배포해서 쓰고, Safari "홈 화면에 추가"로 앱처럼 사용.
- 배포: GitHub Pages 사용 (Vercel은 VPN이 필요해서 사용자가 Pages를 선호). 아래 GitHub 커밋 기능 항목 참고.
- 정리할 것: 잘못 올린 `cnation-ft` 레포의 `claude/clip-saver` 브랜치가 남아 있음(`main` 영향 없음). 사용자가 원하면 삭제.

## GitHub 커밋 기능 (PR #3로 main 머지, GitHub Pages 배포)
- 파일: `index.html`(앱), `apple-touch-icon.png`·`icon-192.png`·`icon-512.png`(임시 아이콘, Python PIL로 생성), `manifest.webmanifest`
- 배포 계획: 사용자가 GitHub Pages로 배포하고 iPhone 홈 화면에 추가해서 사용
- 토큰 저장: localStorage `clipSaver.ghToken`. 홈 화면 웹앱은 Safari와 저장소가 분리되고 ITP 7일 삭제 대상이 아님. `navigator.storage.persist()` 요청. 토큰 입력칸은 username+password 폼이라 iCloud 키체인 자동완성으로 복구 가능. PIN 잠금은 쓰지 않기로 함
- 링크를 아는 타인은 토큰이 없으므로 커밋 불가. 토큰을 코드나 인증 없는 서버 함수에 넣지 말 것
- 레포/브랜치/폴더 선택 기억, 기존 파일 덮어쓰기 전 확인창, 커밋 메시지 비우면 자동 생성
- 2026-10-15 사용자가 iPhone 홈 화면 앱에서 실제 커밋 성공 확인
- 배포 주소: https://civilizednation.github.io/cnation-file/ (GitHub Pages, `main` / root, `.nojekyll`)
- 사용자 토큰: Fine-grained PAT `cnation-file-commit` (All repositories, Contents: Read and write, 만료 2027-01-13 → 만료 전 Regenerate 후 앱에 다시 입력). 처음엔 Repository access가 Public으로 남아 403이 났었음
- 허브 `civilizednation/cnation-app`의 `app-data.js` UTILITY 맨 위에 "cnation File"로 등록됨 (v1.25.2)

## ZIP 업로드 기능
- 📦 ZIP 업로드 패널: ZIP을 고르면 브라우저에서 바로 풀어서 파일 목록 미리보기 → 커밋 1개로 전체 업로드
- 외부 라이브러리 없음: ZIP 목차(central directory)를 직접 파싱, 압축 해제는 `DecompressionStream('deflate-raw')` (iOS 16.4+). 저장(0)·deflate(8) 방식만, ZIP64·암호 ZIP 미지원
- `__MACOSX/`, `.DS_Store`, `._*`, `Thumbs.db`는 제외. UTF-8 플래그 없는 파일명은 UTF-8 시도 후 EUC-KR(Windows 한글 ZIP)로 해석. `..` 경로는 거부
- 모든 파일이 한 폴더 안에 있으면(iPhone 파일 앱에서 폴더 압축) "최상위 폴더 없이 올리기" 체크박스 표시(기본 체크)
- 커밋: Git Data API (ref → base commit/tree → blob 4개씩 병렬 → tree(base_tree) → commit → ref PATCH). 같은 경로는 덮어쓰고 ZIP에 없는 기존 파일은 유지. 확인창에 새 파일/덮어쓰기 개수 표시
- 레포·브랜치·폴더·커밋 메시지는 메인 화면 "📁 커밋 위치" 카드(`#targetFields`, 내용 입력칸 아래·저장 버튼 위)에서 공통으로 선택 → 클립보드 커밋과 ZIP 커밋이 같은 값 사용. 내용 입력칸 높이 22vh. 빈 레포는 ZIP 불가(파일 하나 먼저 커밋 필요)
- Playwright + 모의 API로 테스트(바이너리·한글 내용/파일명, 최상위 폴더, 잘못된 ZIP). 실제 기기 테스트는 아직

설계 메모:
1. 인증: Fine-grained PAT(All repositories, `Contents: Read and write`)를 앱에 한 번 입력해서 localStorage에 저장
2. 레포/브랜치/경로 선택: `GET /user/repos` 등으로 목록을 불러와 드롭다운
3. 업로드+커밋: `PUT /repos/{owner}/{repo}/contents/{path}`. 내용은 base64(UTF-8 안전하게 인코딩), 커밋 메시지 포함. 기존 파일 덮어쓰기는 먼저 GET으로 `sha`를 받아 함께 전송
4. 레포가 Vercel에 연결돼 있으면 커밋 즉시 자동 배포 → "폰에서 복사 → 앱에서 커밋 → 사이트 반영"

보안: 토큰이 기기 localStorage에 남으므로 개인용으로만, 권한은 최소로. 여러 사람이 쓰게 하려면 GitHub OAuth + 서버(Vercel 함수 / Supabase Edge Function)를 거치는 방식으로.

## 사용자 환경
GitHub, Vercel, Firebase, Supabase를 사용. 답변은 한국어로.
