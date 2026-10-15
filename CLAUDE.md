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
- 실제 iPhone 기기 테스트는 아직 안 함.
- 배포: Vercel에서 이 레포를 Import하면 별도 설정 없이 루트 `index.html`이 그대로 배포됨(아직 배포 여부 미확인).
- 정리할 것: 잘못 올린 `cnation-ft` 레포의 `claude/clip-saver` 브랜치가 남아 있음(`main` 영향 없음). 사용자가 원하면 삭제.

## GitHub 커밋 기능 (구현됨, `claude/github-commit` 브랜치)
- 파일: `index.html`(앱), `apple-touch-icon.png`·`icon-192.png`·`icon-512.png`(임시 아이콘, Python PIL로 생성), `manifest.webmanifest`
- 배포 계획: 사용자가 GitHub Pages로 배포하고 iPhone 홈 화면에 추가해서 사용
- 토큰 저장: localStorage `clipSaver.ghToken`. 홈 화면 웹앱은 Safari와 저장소가 분리되고 ITP 7일 삭제 대상이 아님. `navigator.storage.persist()` 요청. 토큰 입력칸은 username+password 폼이라 iCloud 키체인 자동완성으로 복구 가능. PIN 잠금은 쓰지 않기로 함
- 링크를 아는 타인은 토큰이 없으므로 커밋 불가. 토큰을 코드나 인증 없는 서버 함수에 넣지 말 것
- 레포/브랜치/폴더 선택 기억, 기존 파일 덮어쓰기 전 확인창, 커밋 메시지 비우면 자동 생성
- Playwright + 모의 GitHub API로만 테스트함. 실제 기기·실제 API 테스트는 아직

설계 메모:
1. 인증: Fine-grained PAT(대상 레포만, `Contents: Read and write`)를 앱에 한 번 입력해서 localStorage에 저장
2. 레포/브랜치/경로 선택: `GET /user/repos` 등으로 목록을 불러와 드롭다운
3. 업로드+커밋: `PUT /repos/{owner}/{repo}/contents/{path}`. 내용은 base64(UTF-8 안전하게 인코딩), 커밋 메시지 포함. 기존 파일 덮어쓰기는 먼저 GET으로 `sha`를 받아 함께 전송
4. 레포가 Vercel에 연결돼 있으면 커밋 즉시 자동 배포 → "폰에서 복사 → 앱에서 커밋 → 사이트 반영"

보안: 토큰이 기기 localStorage에 남으므로 개인용으로만, 권한은 최소로. 여러 사람이 쓰게 하려면 GitHub OAuth + 서버(Vercel 함수 / Supabase Edge Function)를 거치는 방식으로.

## 사용자 환경
GitHub, Vercel, Firebase, Supabase를 사용. 답변은 한국어로.
