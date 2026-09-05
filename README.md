# 블로그 골격 — GitHub Pages (Jekyll) + 스티비

빌드 도구 없이 GitHub가 직접 렌더링하는 구성. 로컬 설치 없이도 배포 가능.

## 구조

```
site/
├── _config.yml          ← 브랜드명·URL·스티비 폼 URL만 채우면 됨
├── _layouts/default.html, post.html
├── _includes/subscribe.html   ← 구독 CTA (스티비 임베드 자리)
├── assets/style.css
├── index.md  about.md  series.md  corrections.md
├── _posts/              ← 발행 글 (파일명 YYYY-MM-DD-slug.md)
└── drafts/              ← 검증 전 초안 (빌드 제외). 1편 초안 포함
```

## 배포 (10분)

1. GitHub에 새 저장소 생성. 개인 도메인 없이 쓰려면 이름을 `USERNAME.github.io`로.
2. 이 폴더의 내용을 저장소 루트에 커밋·푸시.
3. 저장소 Settings → Pages → Source: **Deploy from a branch**, Branch: `main` / `(root)` → Save.
4. 1–2분 후 `https://USERNAME.github.io` 에서 확인.
5. `_config.yml`의 `url`, `title`, `email` 수정 후 다시 푸시.

커스텀 도메인: Settings → Pages → Custom domain에 입력 → DNS에 CNAME(`USERNAME.github.io`) 추가 → "Enforce HTTPS" 체크. 저장소 루트에 `CNAME` 파일이 자동 생성됨.

## 스티비 연결

1. stibee.com 가입 → 주소록 생성 → 구독 폼 만들기.
2. 구독 폼의 **링크 URL**을 `_config.yml`의 `stibee_form_url`에 입력 → 헤더·글 하단 CTA가 자동 표시됨.
3. (선택) 임베드 폼: 스티비 "웹사이트에 삽입" 코드를 `_includes/subscribe.html`에 붙여 넣기.
4. 발행 흐름: 글 푸시 → 스티비에서 `#뉴스레터` 모드로 만든 본문 + 글 링크로 이메일 예약 발송.

## 글 발행 규칙

- 초안은 `drafts/`에 두고, 검증표 전 항목 체크 후 `_posts/YYYY-MM-DD-slug.md`로 이동.
- front matter `verified: true`로 바꾸면 배지가 "실기 검증 완료"로 변경.
- 정정 시 글의 `corrections:` 배열과 `corrections.md` 표에 같이 기록.

## 로컬 미리보기 (선택)

```
gem install bundler && bundle install && bundle exec jekyll serve
```

## 비용

GitHub Pages 무료(공개 저장소), 스티비 구독자 500명까지 무료, 도메인 연 1–2만원.
