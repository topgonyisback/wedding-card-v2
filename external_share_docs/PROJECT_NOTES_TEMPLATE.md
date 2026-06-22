# 모바일 청첩장 프로젝트 기록

이 문서는 외부 공유용으로 정리한 모바일 청첩장 프로젝트 기록 템플릿이다.
실제 저장소 주소, 로컬 경로, Firebase 프로젝트 정보, 이미지 파일명은 예시값으로 치환했다.

## 기본 정보

- 프로젝트 위치: `<LOCAL_PROJECT_PATH>`
- 배포 URL: `https://<GITHUB_USER>.github.io/<REPOSITORY_NAME>/`
- GitHub 저장소: `https://github.com/<GITHUB_USER>/<REPOSITORY_NAME>`
- 메인 파일: `index.html`
- 주요 이미지 폴더: `assets/`, `assets/optimized/`
- 방명록 DB: Firebase Cloud Firestore

## 전체 구조

현재 프로젝트는 정적 HTML 기반 모바일 청첩장이다.

- HTML, CSS, JavaScript가 메인 파일에 모여 있다.
- GitHub Pages로 정적 호스팅한다.
- 축하 메시지 방명록만 Firebase Cloud Firestore를 사용한다.
- 이미지와 영상은 로컬 asset 폴더에 두고 WebP/MP4를 중심으로 사용한다.

## 주요 섹션

1. Intro Hero
   - 첫 화면의 메인 히어로.
   - 스크롤에 따라 중앙 이미지와 주변 이미지가 움직이는 인터랙션이 있다.
   - 이 영역은 레이아웃이 민감하므로 다른 작업 중 불필요하게 수정하지 않는다.

2. Greeting
   - 초대 인사 문구 영역.
   - 스크롤하면서 3개의 문장이 순차적으로 노출된다.
   - GSAP ScrollTrigger를 사용한다.

3. Couple
   - 신랑, 신부 소개 영역.
   - 1:1 프레임 안에 영상 또는 이미지를 배치할 수 있다.

4. Our Story
   - 두 사람이 만나게 된 과정을 가로 스크롤 카드로 보여준다.
   - 모바일에서는 GSAP pin을 제거하고 일반 가로 스크롤만 사용한다.

5. Our Moments
   - 웨딩 사진 갤러리.
   - 가로 스크롤 카드와 클릭 확대 팝업을 제공한다.

6. D-day / Venue
   - 예식 날짜, 카운트다운, 예식장 소개 영역.
   - 위치 상세 보기 버튼으로 지도/오시는 길 팝업을 열 수 있다.

7. Guide
   - 식사, 주차, 포토부스 안내.
   - 가로 스크롤 카드 구조.

8. Account
   - 마음 전하기 버튼을 누르면 전체 화면 팝업으로 계좌 정보를 보여준다.
   - 계좌 복사 시 하단 통합 토스트를 사용한다.

9. Guestbook
   - 축하 메시지를 남기는 영역.
   - Firebase Firestore의 `guestbook` 컬렉션에 저장한다.

10. Outro
   - 마지막 공유/링크 복사용 히어로 영역.
   - OG 이미지와 공유 문구를 함께 관리한다.

## Firebase 요약

Firestore 컬렉션:

```text
guestbook
```

문서 예시:

```json
{
  "name": "홍길동",
  "message": "두 분의 결혼을 축하합니다.",
  "createdAt": "2026-06-08T00:04:59.200Z"
}
```

주의:

- `createdAt`은 서버 타임스탬프 대신 ISO 문자열로 저장할 수 있다.
- Firestore 보안 규칙은 읽기와 생성만 허용하고, 수정/삭제는 막는다.
- 이름과 메시지 길이 제한을 규칙에 포함한다.

## OG / 링크 공유

링크 공유 썸네일과 문구는 HTML의 meta 태그에서 관리한다.

대표 이미지 예시:

```text
https://<GITHUB_USER>.github.io/<REPOSITORY_NAME>/assets/og-thumbnail.webp
```

필수 태그:

```html
<meta property="og:title" content="모바일 청첩장" />
<meta property="og:description" content="두 사람의 결혼식에 초대합니다." />
<meta property="og:image" content="https://<GITHUB_USER>.github.io/<REPOSITORY_NAME>/assets/og-thumbnail.webp" />
<meta property="og:url" content="https://<GITHUB_USER>.github.io/<REPOSITORY_NAME>/" />
```

## 배포 흐름

```bash
git status --short
git add index.html assets/optimized
git commit -m "Update wedding invitation"
git push origin main
```

GitHub Pages 반영에는 보통 약간의 시간이 걸린다.

## 작업 시 주의사항

- Intro Hero는 가장 민감한 영역이므로 불필요하게 수정하지 않는다.
- Our Story, Our Moments는 모바일에서 pinning을 사용하지 않는다.
- 이미지 추가 시 가능하면 WebP로 변환하고 `assets/optimized/`에 둔다.
- 외부 라이브러리와 Firebase SDK는 초기 로딩을 피하고 필요한 시점에 로드한다.
- 새 이미지가 git에 추적되지 않은 상태인지 항상 `git status --short`로 확인한다.
