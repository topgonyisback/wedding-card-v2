# GitHub Pages 배포 가이드

이 문서는 모바일 청첩장 정적 사이트를 GitHub Pages로 배포하기 위한 외부 공유용 템플릿이다.
실제 저장소 주소와 파일명은 예시값으로 치환했다.

## 배포 정보

- 저장소: `https://github.com/<GITHUB_USER>/<REPOSITORY_NAME>`
- 브랜치: `main`
- 배포 방식: GitHub Pages
- 배포 URL: `https://<GITHUB_USER>.github.io/<REPOSITORY_NAME>/`
- 로컬 프로젝트: `<LOCAL_PROJECT_PATH>`

## 기본 배포 흐름

수정 후 아래 순서로 진행한다.

```bash
git status --short
git add index.html
git commit -m "Update wedding invitation"
git push origin main
```

이미지나 문서도 함께 배포해야 하면 해당 파일도 같이 추가한다.

```bash
git add index.html assets/optimized/example-image.webp
git commit -m "Update assets"
git push origin main
```

## GitHub Pages 설정

GitHub 저장소에서:

1. Settings 이동
2. Pages 메뉴 선택
3. Source를 `Deploy from a branch`로 설정
4. Branch를 `main`으로 설정
5. Folder를 `/root` 또는 프로젝트 구조에 맞게 선택
6. Save

반영 후 아래 URL로 확인한다.

```text
https://<GITHUB_USER>.github.io/<REPOSITORY_NAME>/
```

## 캐시 확인

GitHub Pages는 반영에 시간이 조금 걸릴 수 있다.

수정이 바로 안 보이면 캐시 우회를 위해 쿼리스트링을 붙여 확인한다.

```text
https://<GITHUB_USER>.github.io/<REPOSITORY_NAME>/?v=check1
```

단, 사용자에게 공유할 최종 URL은 쿼리스트링 없는 기본 주소를 권장한다.

```text
https://<GITHUB_USER>.github.io/<REPOSITORY_NAME>/
```

## OG 이미지 확인

링크 공유 썸네일은 캐시가 오래 남을 수 있다.

예시:

```text
https://<GITHUB_USER>.github.io/<REPOSITORY_NAME>/assets/og-thumbnail.webp
```

OG 이미지가 바뀌지 않으면:

- 이미지 파일명을 새로 바꾼다.
- meta 태그의 `og:image` 경로도 함께 바꾼다.
- 카카오/페이스북/메신저의 공유 디버거에서 캐시를 갱신한다.

## Git 상태 확인

배포 전 항상 확인한다.

```bash
git status --short
```

주의:

- 의도하지 않은 이미지나 임시 파일이 함께 올라가지 않도록 확인한다.
- 새 이미지가 필요하면 명확히 확인한 뒤 add한다.
- 문서만 배포할 때는 사이트 파일을 불필요하게 같이 수정하지 않는다.

## 로컬 테스트

간단한 정적 서버:

```bash
python3 -m http.server 4186
```

브라우저에서:

```text
http://localhost:4186/
```

모바일 테스트:

- 같은 Wi-Fi에 연결
- Mac의 로컬 IP 확인
- 모바일 브라우저에서 `http://<LOCAL_IP>:4186/` 접속

## 배포 전 체크리스트

- PC 첫 화면 히어로 정상
- 모바일 첫 화면 히어로 정상
- GNB 클릭 이동 정상
- Our Story 가로 스크롤 정상
- Our Moments 가로 스크롤 및 팝업 정상
- Guestbook 저장/조회 정상
- Account 복사 토스트 정상
- OG 썸네일 정상

## 문제 해결

### 404가 뜨는 경우

확인할 것:

- GitHub Pages가 켜져 있는지
- `main` 브랜치에 `index.html`이 있는지
- 저장소명이 URL과 일치하는지
- Pages 반영 시간이 지났는지

### 수정했는데 안 보이는 경우

확인할 것:

- push가 완료되었는지
- GitHub Actions 또는 Pages 배포가 완료되었는지
- 브라우저 캐시를 우회했는지
- URL 뒤에 `?v=check1`을 붙여 확인했는지

### Firebase 방명록 오류

확인할 것:

- Firestore 규칙이 배포되었는지
- 프론트 payload가 규칙과 일치하는지
- `createdAt` 타입이 규칙과 맞는지
- 컬렉션명이 `guestbook`인지
